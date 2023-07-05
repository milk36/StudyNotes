<!-- markdownlint-disable MD033 -->
# 荒古备忘

## 部署流程

ubuntu 22.04 LTS
```sh
sudo apt install -y protobuf-compiler protobuf-c-compiler libreadline-dev autoconf git subversion telnet netcat libcurl4-openssl-dev libssl-dev gawk
sudo apt install build-essential
sudo apt install python3
sudo ln -s /bin/python3 /bin/python
```

### [Openssl 1.1.1安装](https://www.cnblogs.com/tinywan/p/13961228.html)

Ubuntu 22.04 LTS Openssl为 3.0, 因此 `skynet` 编译无法通过

## 调试小技巧
  
* `print(debug.traceback())` 打印堆栈调用信息
* `table.dump()` 打印table 将表dump成字符串(便于人类阅读的格式,支持环状引用) 此为table扩展 对应脚本 `gg.base.util.table`
* `gg.json.encode()` 将对象打包成json对象

## 启动加载顺序

### gameserver -> src -> app -> game -> main.lua

* 启动文件加载顺序

  ```lua
  --main.lua 脚本
  skynet.init(function ()
    local ok,err = pcall(game.init) 
    end)
  ...

  -- game.lua脚本初始化
  require "app.game.init"
  ...
  function game.init()
    skynet.memlimit(1024*1024*1024) --设定当前服务最多可以使用多少字节的内存 1G
    skynet.register(".game") --注册game服务
    ...
    logger.init() --启动日志服务
    gg.init()     --gg相关模块初始化
    ...
  end

  --启动相关服务
  skynet.start(function ()
      local ok,err = pcall(game.start) --启动集群,tcp,kcp,websocket等网络服务
      ...
      require "app.game.world"
      gg.world = gg.class.cworld.new()
      skynet.name('.world', skynet.newservice("app/world/world")) --启动场景管理器服务?
      skynet.name('datacenter', skynet.newservice("app/world/datacenter")) --启动数据中心服务
      if gg.isgameserver() then
          skynet.name('rankor', skynet.newservice("app/world/services/rankor")) --启动排行榜服务
          skynet.name('teamor', skynet.newservice("app/world/services/teamor")) --启动组队服务
          skynet.call('teamor', 'lua', 'init')
          gg.world:createspaceglobal() --src/app/game/world.lua 创建全局空间?
      end
  end)
  ```

  1. `require "app.game.init"`加载内容 -> gameserver > src > app > game > init.lua
  
      ```lua
      require "gg.init" --基础模块加载
      ...
      require "app.game.gg.init" --业务模块加载
      ```

      1. `require "gg.init"` -> gameserver > src > gg > init.lua

          ```lua
          -- 
          require "gg.init"

          -- gg > init.lua
          gg = gg or {}   --gg后续将关联所有模块引用(脚本指针)
          require "gg.base.init"

          -- gg > base > init.lua
          skynet = require "skynet.manager" --注册skynet管理器
          ...  --gg模块下相关的工具模块引用
          
          gg.isgameserver = function ()
            return skynet.getenv('type') == 'gameserver'  --对应到配置文件中 type = "gameserver"  服务器类型
          end
          ```

      1. `require "app.game.gg.init"` -> gameserver > src > app > game > gg > gg.lua

          ```lua
          --定义gg初始化,相关模块class构建等操作
          function gg.init()
            gg.profile = gg.class.cprofile.new()
            gg.timer = gg.class.ctimer.new()
            gg.sync = gg.class.csync.new()
            gg.actor = gg.class.cactor.new()

            gg.dbmgr = gg.class.cdbmgr.new()
            gg.savemgr = gg.class.csavemgr.new()
            gg.timectrl = gg.class.ctimectrl.new()
          end
          ```

  2. `gg.init()` 模块引用初始化

      ```lua
      function gg.init() --初始相关模块对象 dbmgr;playermgr;
      ```

## class 对象

### class.lua

* gg > base > class.lua 脚本文件
  
  给lua oop提供原语class,支持热更新，支持父类热更新直接
* gg.class 原表的申明
* 使用方式
  
  ```lua
  citem = class("citme") --默认会调用ctor或init函数
  ```

* class接口测试

  ```lua
  print(typename(gg.thistemp))
  print(isinstance(gg.thistemp,gg.class.cthistemp))
  ```

### 申明class

```lua
local cdatabaseable = gg.class.cdatabaseable --获取cdatabaseable Class
local ctoday = class("ctoday",cdatabaseable) --新建ctody对象 继承自 cdatabaseable

--或者
cthisweek = class("cthisweek",gg.class.ctoday)
```

### ctor 与 init

`ctor` 构造函数用于处理父类继承关系相关的数据初始,内部也有可能回调用`init`初始函数

`xxxclass.new(...)` 会优先调用`ctor`,

## watchdog gate agent 网络连接接入

1. gate主要负责的是client连接创建与断开以及接受到消息转发给agent。

1. watchdog主要负责gate的创建，agent的创建与退出。

1. agent主要负责接受gate转发的请求，处理业务，然后直接把应答发送给client。

### 网络消息处理流程(以tcp为例)

### tcp_gate

`game` `actor` `tcp_gate` `actor/client`

* src/app/game/game.lua `watchdog` 启动 `tcp_gate, kcp_gate, websocket_gate` 服务并建立关联

  ```lua
  function game.start()
  ...
  gg.actor:start() --启动actor
  local address = skynet.self()
  -- 多久不活跃会主动关闭套接字(1/100秒为单位)
  local gate_conf = {
      watchdog = address,
      ...
  }
  ...
  local tcp_gate
  local kcp_gate
  local websocket_gate
  local tcp_port = skynet.getenv("tcp_port")
  if tcp_port then
      gate_conf.port = tcp_port
      tcp_gate = skynet.uniqueservice("gg/service/gate/tcp")
      skynet.call(tcp_gate,"lua","open",gate_conf)
  end
  ...
  gg.actor.client.tcp_gate = tcp_gate
  gg.actor.client.kcp_gate = kcp_gate
  gg.actor.client.websocket_gate = websocket_gate
  gg.actor.client:open() --gameserver/src/app/game/client/client.lua --> cclient:open()
  _print("client:open")
  end
  ```

* src/gg/service/gate/tcp.lua
  
  ```lua
  local connection = {} --保存连接相关信息

  function CMD.open(conf)
    -- 多长时间(单位:1/100秒)未收到客户端协议,则主动断开该客户端连接
    timeout = conf.timeout or 0 --socket_timeout = 6000 /100 s 超时时间为60秒
    watchdog = assert(conf.watchdog) --watchdog 为 src/app/game/game.lua 中嵌入的actor
    ...
    local id = assert(socket.listen(ip,port))
    skynet.error("Tcp listen on",ip,port)
    socket.start(id,function (linkid,addr) --启动TCP 并监听端口
    ...
  end

  --建立连接
  function handler.onconnect(linkid,addr)
    ...
    skynet.send(watchdog,"lua","client","onconnect","tcp",linkid,addr) --watchdog 为 src/app/game/game.lua
    ...
  end

  --接收消息
  function handler.onmessage(linkid,msg)
    ...
    local message = codecobj:unpack_message(msg) --解包消息
    local address = forward_protos[message.cmd] or watchdog
    skynet.send(address,"lua","client","onmessage",linkid,message) --转发到client
  end
  ```

  ```sh
  01/12/23 14:34:48.99 [:0000000c] op=onconnect,linktype=tcp,linkid=38,addr=127.0.0.1:38082
  ```

### actor

* src/app/common/actor/actor.lua 初始构建 `cclient , ccluster , cinternal` 等
  
  ```lua
  function cactor:init()
    self.client = gg.class.cclient.new() --客户端协议
    self.cluster = gg.class.ccluster.new() --集群协议
    self.internal = gg.class.cinternal.new() --节点内部协议
    ...
  end
  ```

* src/app/common/actor/actor.lua 分发消息 <a id="actor_dispatch"></a>
  
  ```lua
  --启动
  function cactor:start(callback)
    skynet.dispatch("lua",function (...)
        self:dispatch(...) --cactor:dispatch()
    end)
  end

  function cactor:dispatch(session,source,typ,...)
    local cmd = self:extract_cmd(typ,...) --解析指令
    ...
    if cmd then
        --性能分析 帮助统计一个消息处理使用的系统时间
        local profile = gg.profile
        profile.cost[typ] = profile.cost[typ] or {__tostring=tostring,}
        ok,err = profile:stat(profile.cost[typ],cmd,self.onerror,self._dispatch,self,session,source,typ,...)
    else
        ok,err = xpcall(self._dispatch,self.onerror,self,session,source,typ,...)
    end
    ...
  end
  
  --分发消息
  function cactor:_dispatch(session,source,typ,...)
      --skynet.trace()
      if typ == "client" then
          -- 客户端消息
          self.client:dispatch(session,source,...) --actor/client
      elseif typ == "cluster" then
          -- 集群(服务器间）消息
          self.cluster:dispatch(session,source,...)
      elseif typ == "internal" then
          -- 同节点actor间通信
          self.internal:dispatch(session,source,...)
      elseif typ == "gm" then
          -- debug_console发过来的gm消息
          self.gm:dispatch(session,source,...)
      elseif typ == "world" then
          --转发从map到来的数据
          gg.playerlogic:sendtoclient(...)
      end
  end
  ```

  extract_cmd 解析的指令格式:

  ```sh
  typ: client msg data:  onconnect       tcp     38      127.0.0.1:38082 #linkid = 38
  typ: client msg data:  onhandshake     tcp     38      127.0.0.1:38082 OK
  typ: client msg data:  onmessage       38      table: 0x7f0a466652c0
  onmessage:+cmd [C2GS_CheckToken]
  +session [0]
  +args+forward [EnterGame]
  |    +account [111]
  |    +token [debug]
  |    +version [99.99.99]
  +type [1]
  ...
  typ: client msg data:  onclose 38
  ```

### actor / cclient

* src/app/common/actor/client.lua

  ```lua
  --扩展cclient
  local cclient = gg.class.cclient

  --注册关联cmd与协议handler
  function cclient:register(cmd,handler)
  end

  --给客户端发送“请求”消息
  cclient.sendpackage = cclient.send_request

  --- 调度网关收到的客户端消息,如连接、关闭、收到数据等
  function cclient:dispatch(session,source,cmd,...)
      if cmd == "onconnect" then
          self:onconnect(...)
      elseif cmd == "onhandshake" then
          self:onhandshake(...)
      elseif cmd == "onclose" then
          self:onclose(...)
      elseif cmd == "onmessage" then
          self:onmessage(...)
      elseif cmd == "http_onmessage" then
          self:http_onmessage(...)
      elseif cmd == "slaveof" then
          self:slaveof(...)
      end
  end

  --- 收到客户端连接
  --@param[type=string] linktype 连接类型,如tcp/kcp/websocket
  --@param[type=int] linkid 连接ID
  --@param[type=string] addr 连接地址
  function cclient:onconnect(linktype,linkid,addr)
      local linkobj = gg.class.clinkobj.new(linktype,linkid,addr) --src/gg/client/linkobj.lua 连接数据
      self:addlinkobj(linkobj)
  end

  --- 收到客户端消息
  --@param[type=int] linkid 连线ID
  --@param[type=table] message 消息
  function cclient:onmessage(linkid,message)
    ...
    if message.type == 1 then
        -- request
        local handler = self.cmd and self.cmd[cmd] --cclient:register(cmd,handler) 注册关联cmd与协议handler
        if handler then
            handler(player,message) --
        elseif self.forwardworld then
            self:forwardworld(player, message)

        end
    else
    ...
  end
  ```

* gg/client/client.lua
  
  ```lua
  --- 让gate转发协议到指定服务
  --@param[type=int] linkid 连接ID
  --@param[type=string] proto 协议名
  --@param[type=int|string] address 服务地址
  function cclient:forward(linkid,proto,address)
      if self.tcp_gate then
          skynet.send(self.tcp_gate,"lua","forward",linkid,proto,address)
      end
      if self.websocket_gate then
          skynet.send(self.websocket_gate,"lua","forward",linkid,proto,address)
      end
      if self.kcp_gate then
          skynet.send(self.kcp_gate,"lua","forward",linkid,proto,address)
      end
  end
  ```

## protobuf 加载

### 配置文件的加载

* pb文件路径:`tools/proto/protobuf`
* protobuf生成脚本文件: `tools/proto/protobuf/run_on_change.sh`
* `src/app/config/custom.lua`

  ```lua
  -- proto
   
  proto = {
      type = "protobuf",
      pbfile = "src/proto/protobuf/all.pb",
      idfile = "src/proto/protobuf/message_define.lua",
  },
  ```

* `src/gg/codec/codec.lua` 加载协议文件/解包协议/封包协议
  * `src/gg/codec/protobuf.lua`

## handler 协议逻辑

### gameserver各个模块协议handler的注册与关联

* src/app/game/client/client.lua

  ```lua
  function cclient:register_module(name,module,prefix) 
    prefix = prefix or "C2GS"
    self[name] = module
    for proto,handler in pairs(module[prefix]) do
        self:register(prefix.."_"..proto,handler) --actor/client:register(cmd,handler)
    end
  end

  --模块协议handler注册
  function cclient:open()
    ...
    self:register_http("/api/rpc",require "app.game.client.http.api.rpc")
    self:register_module("login",require "app.game.client.login", nil)
    self:register_module("map",require "app.game.client.map" , nil)
    self:register_http("/api/roleonline",require "app.game.client.http.api.roleonline")
    self:register_http("/api/roleonlinechannel",require "app.game.client.http.api.roleonlinechannel")
    self:register_http("/api/roleprops",require "app.game.client.http.api.roleprops")
    self:register_http("/api/sendmail",require "app.game.client.http.api.sendmail")
    self:register_http("/api/itemsynchron",require "app.game.client.http.api.itemsynchron")
    self:register_http("/api/kickout",require "app.game.client.http.api.kickout")
    self:register_http("/api/modifyname",require "app.game.client.http.api.modifyname")        
    self:register_module("team",require "app.game.client.team" , nil)
    self:register_module("playerhandle",require "app.game.client.playerhandle" , nil)

    ...
  end

  --转发消息到world服务
  function cclient:forwardworld(player, message)
    local s, e = string.find(message.cmd, "C2GS_World")
    if -1 ~= s and player.spacemailbox then
        gg.world:sendtonode(player.pid, player.spacemailbox, message.cmd, message)
    end
  end
  ```

* src/app/common/actor/client.lua
  
  ```lua
  --注册协议与handler函数
  function cclient:register(cmd,handler) 
      if not self.cmd then
          self.cmd = {}
      end
      self.cmd[cmd] = function (linkobj, message)
          if not linkobj.cmdrequestlist then
              linkobj.cmdrequestlist = {}
          end
          if not linkobj.cmdrequestlist[message.cmd] then
              linkobj.cmdrequestlist[message.cmd] = true
              handler(linkobj, message)
              linkobj.cmdrequestlist[message.cmd] = nil
          end
      end
  end

  --注册非安全认证可访问协议指令
  function cclient:register_unsafe(cmd,handler)
    if not self.unsafe_cmd then
        self.unsafe_cmd = {}
    end
    self.unsafe_cmd[cmd] = handler
  end

  --注册http指令
  function cclient:register_http(cmd,handler)
    if not self.http_cmd then
        self.http_cmd = {}
    end
    self.http_cmd[cmd] = handler
  end
  ```

  注册协议列表:

  ```sh
  cmd : C2GS_RoleList
  cmd : C2GS_CreateRole
  cmd : C2GS_ExitGame
  cmd : C2GS_EnterGame
  cmd : C2GS_ModifyName
  cmd : C2GS_Ping
  cmd : C2GS_GetProgressState
  cmd : C2GS_CheckToken
  ...
  ```

### loginserver http API 模块handler

* src/app/game/client/client.lua

  ```lua
  function cclient:open()
    self:register_http("/api/rpc",require "app.game.client.http.api.rpc")
    self:register_http("/api/app/add",require "app.game.client.http.api.app.add")

    self:register_http("/api/account/register",require "app.game.client.http.api.account.register")
    self:register_http("/api/account/login",require "app.game.client.http.api.account.login")
    self:register_http("/api/account/checktoken",require "app.game.client.http.api.account.checktoken")
    self:register_http("/api/account/role/add",require "app.game.client.http.api.account.role.add")
    self:register_http("/api/account/role/del",require "app.game.client.http.api.account.role.del")
    self:register_http("/api/account/role/recover",require "app.game.client.http.api.account.role.recover")
    ...
  end
  ```

## httpd服务

以登陆服为例

### httpd的启动

以下涉及的脚本均为`loginserver`节点

* `src/app/game/game.lua`

  ```lua
  function game.start()
  ...
      local gate_conf = {
          --watchdog = address,
          service_name = "app/game/httpd_main", --服务名
      }
      local http_port = skynet.getenv("http_port")
      if http_port then
          gate_conf.port = http_port --http端口
          httpd.start(gate_conf)
      end
  ...
  end
  ```

* `src/app/common/http/httpd.lua` 对于skynet.httpd的封装

  ```lua
  function httpd.start(conf)
  ...
    local service_name = assert(conf.service_name)
    skynet.start(function ()
        for i=1,agent_num do --默认开启16个httpd服务
            httpd.webagents[i] = skynet.newservice(service_name) --service_name = "app/game/httpd_main"
        end
        local balance = 1 --负载均衡
        local id = socket.listen(ip,port) --监听端口
        skynet.error(string.format("Listen web port %s:%s",ip,port))
        socket.start(id , function(linkid, addr)
            local client_ip,client_port = string.match(addr,"([^:]+):(%d+)")
            skynet.send(httpd.webagents[balance], "lua","start",watchdog,linkid,client_ip,client_port,msg_max_len) --将数据发送到对应httpd服务
            balance = balance + 1
            if balance > agent_num then
                balance = 1
            end
        end)
    end)
  end
  ```

### httpd_main 解析协议

* `src/app/game/httpd_main.lua` httpd与业务模块
  
  ```lua
  skynet.start(function ()
    skynet.dispatch("lua",function (session,source,cmd,...) --接收发送给本服务的消息 : "lua","start"
        local func = assert(handler[cmd],cmd)
        func(...)
    end)
  end)

  function handler.start(watchdog,linkid,ip,port,msg_max_len)
    socket.start(linkid)
    local code, url, method, header, body = httpd.read_request(sockethelper.readfunc(linkid), msg_max_len)
    if code then
        if code ~= 200 then
            response(linkid, code)
        else
          local agent = {
                linkid = linkid,
                ip = ip,
                port = port,
                method = method, --method 是 "GET" "POST" 等。
            }
          if watchdog then
                ok = pcall(skynet.call,watchdog,"lua","service","http_onmessage",agent,uri,header,query,body)
          else
              ok = xpcall(gg.actor.client.http_onmessage,gg.onerror or debug.traceback,gg.actorclient,agent,uri,header,query,body) --actor/client->http_onmessage处理相关模块协议
          end
        end
    else
      ...
    end
    socket.close(linkid) --处理完数据后关闭socket
  end
  ```

### http协议分发到模块

* `src/app/common/actor/client.lua` 模块协议分发
  
  ```lua
  function cclient:http_onmessage(linkobj,uri,header,query,body)
    local handler = self.http_cmd and self.http_cmd[uri]
    if handler then
        local func = handler[linkobj.method] --method 是 "GET" "POST" 等。
        if func then
            func(linkobj,header,query,body)
        else
            -- method not implemented
            httpc.response(linkobj.linkid,501)
        end
    else
        -- not found
        httpc.response(linkobj.linkid,404)
    end
    skynet.ret(nil)
  end
  ```

* `src/app/game/client/http/api/account/login.lua` 以http登陆请求为例

  ```lua
  local handler = {}

  function handler.exec(linkobj,header,args)
    ... --进行登陆相关验证
  end

  function handler.POST(linkobj,header,query,body) --POST调用
    local args = cjson.decode(body)
    handler.exec(linkobj,header,args)
  end
  ```

## 登陆相关协议

* `C2GS_CheckToken`
* `C2GS_EnterGame`

### src/app/game/gg/loginserver.lua 用于gameserver与loginserver进行http api请求

  ```lua
  ---校验token
  --@param[type=string] account 账号
  --@param[type=string] token 认证token
  --@return[type=int] status 返回的状态码
  --@return[type=string] response 回复数据
  function cloginserver:checktoken(account,token)
      local url = "/api/account/checktoken"
      local req = self:encode_request({
        appid = self.appid,
        account = account,
        token = token,
      })
      return self:decode_response(self:post(url,req))
  end

  ---获取角色
  --@param[type=int] roleid 角色ID
  --@return[type=int] status 返回的状态码
  --@return[type=string] response 回复数据
  function cloginserver:getrole(roleid)
      local url = "/api/account/role/get"
      ...
  end

  ---更新角色
  --@param[type=int] roleid 角色ID
  --@param[type=table] role 角色数据
  --@return[type=int] status 返回的状态码
  --@return[type=string] response 回复数据
  function cloginserver:updaterole(roleid,role)
      local url = "/api/account/role/update"
      ...
  end
  ```

### Player 登陆后数据加载

* `src/app/game/client/login.lua`
  
  ```lua
  function C2GS.EnterGame(linkobj,message)
    ...
    local ok,errmsg = gg.sync:once_do(id,netlogin._entergame,linkobj,message)
  end

  function netlogin._entergame(linkobj,message)    
    --判断重复登陆,顶号登陆,黑名单等逻辑
    ...
    local player = gg.playermgr:getplayer(pid)
    if player then
      ...
    else
      player = gg.playermgr:recoverplayer(pid) --数据库查询玩家信息 判断是否已经创建过账号
    end
    ...
    gg.playermgr:bind_linkobj(player,linkobj) --绑定linkobj与player
    gg.actor.client:sendpackage(linkobj,"GS2C_EnterGameStart", {...}) --通知client进入游戏
    player:entergame(replace) --加载player数据
    if replace then
        player:fireevent(gg.commondefine.EVENT_PLAYER_RECONNECT, replace)
    else
        player:fireevent(gg.commondefine.EVENT_PLAYER_LOGIN, replace) --触发事件
    end
  end
  ```

* `src/app/game/player/playermgr/playermgr.lua` player管理器
  
  ```lua
  function cplayermgr:recoverplayer(pid) --从数据库中查询
    ...
  end

  ```

  * `src/app/game/player/playermgr/kuafu.lua` 扩展`playermgr.lua` 主要涉及跨服相关的逻辑

    ```lua
    -- 根据玩家ID获取<当前所在服,是否在线>
    function cplayermgr:route(pid) --loginserver -> /api/account/role/get 获取中心服的角色数据
    end
    ```

* `src/app/game/player/player.lua` player数据结构
* `src/app/game/player/playerlogic.lua` 注册事件消费函数
* `src/app/game/gg/commondefine.lua` 事件常量

## gameserver 刷新状态到loginserver

### gameserver -> src/app/game/server.lua

  ```lua
  ---启动定时器
  function cserver:starttimer()
    local interval = 10 --间隔十秒执行
    gg.timer:timeout("server.on_tick",interval,function ()
        self:starttimer() --自旋调用
    end)
    pcall(self.on_tick,self) --执行on_tick函数
  end

  function cserver:on_tick()
    ...
    local status,response = gg.loginserver:updateserver(serverid,server) --将当前gameserver状态刷新到loginserver
    --/api/account/server/update
    ...
  end
  ```

### loginserver -> src/app/game/client/http/api/account/server/update.lua

  api url:`/api/account/server/update`

## app.world.handlers.handlers.lua 场景相关信息

### handlers.lua 协议处理器

* handlers的初始化 `gameserver -> src -> app -> world -> init.lua`
  
  ```lua  
  require "app.world.handlers.handlers"
  gg.handlers = gg.class.chandlers.new()
  ```

* handlers的热更 `handlers.lua`

  ```lua
  function __hotfix(module)
        gg.handlers = gg.class.chandlers()
  end
  ```

## dbmgr 数据库管理器

### gameserver 数据库配置的加载

* 数据库配置文件: `src/app/config/custom.lua`
  
  ```lua
  ...
  db_config = {
        --db_is_cluster = true,
        db_type = "mongodb",
        db = skynet.getenv("appid") or "game", --数据库名 src/app/config/common.config 配置项: appid = "arpg"
        rs = {
            -- {host = "127.0.0.1",port = 26000,username=nil,password=nil,authmod="scram_sha1",authdb="admin"},
            {host = "127.0.0.1",port = 28018,username="mongo-admin",password="123456",authmod="scram_sha1",authdb="admin"},
        }
    },
  db_nodes = {
    mongodb = {xxx},
    redis = {xxx}
  }
  ...
  ```

* src/app/common/gg/dbmgr.lua db管理器对应的数据库

  ```lua
  function cdbmgr:getdb(db_id)
    db_id = db_id or skynet.getenv("id")
    local ok,obj = gg.sync:once_do(db_id,function ()
        return self:_getdb(db_id) --配置项为: id = "gameserver_lyh" 服务器ID
    end)
    ...
    local conf = obj.conf -- conf.db = appid ="arpg" 数据库名
    ...
  end

  function cdbmgr:get_db_conf(db_id) --
    -- db_nodes中需要存放所有需要连接的db的配置,如果服务器只需要连本服数据库,
    -- 则可以不提供db_nodes,以第二种形式提供本服数据库配置即可
    if db_id == skynet.getenv("id") then --默认数据库
        return skynet.getenv("db_config")
    end
    local db_nodes = skynet.getenv("db_nodes")
    if db_nodes then
        return db_nodes[db_id]
    end
  end
  ```

  * redis和mongodb的区分
  
     `local db = gg.dbmgr:getdb()` 默认使用 `db_config` 配置项

     `local db = gg.dbmgr:getdb('redis')` 会使用 `db_nodes` 对应的配置项

## 游戏配置数据

* `src/app/world/protoconfig`

## world 地图场景管理服务

* `src/app/world/commondefine.lua` world常量文件

### 进入场景

* `C2GS.Map_EnterSpace`

#### game 服务

* `src/app/game/client/map.lua`
  
  ```lua
  function C2GS.Map_EnterSpace(player,message)
    ...
    spacedata = {
            spaceid = spaceid,
            spaceguid = snowflake.uuid(skynet.getenv("index")), --雪花算法生成 全局id(guid)
            nodename = "cnodepersonal"
        }
    ...
    skynet.fork(function()
      local ret = gg.world:teleportspace(player, spacedata, celldata)
    end)
    ...
  end
  ```

* `src/app/game/world.lua`

  ```lua
  function cworld:teleportspace(player, spacedata, celldata) --传送到场景
    ...
    local spacemailbox = self:getspace(spacedata.spaceguid)
    if not spacemailbox then
        spacemailbox = self:createspace( spacedata) --如果场景服务为空,则创建一个新场景服务
        if not spacemailbox then
            return
        end
    end
    local mysmb = player.spacemailbox
    if mysmb and mysmb.spaceguid ~= spacemailbox.spaceguid then
        self:leavespace(player,'teleportspace') --离开旧场景 调用: space:leavespace( entityguid, reason )
    end
    local ret, spacemailbox = self:enterspace(player, spacemailbox, spacedata, celldata) --进入场景 space:enterspace -> space:enterspaceimp
    return spacemailbox
  end

  function cworld:getworldserver() --获取world服务 对应到配置文件: src/app/config/nodes.lua
    local nodes = skynet.getenv("nodes") or {}
    local servers = {}
    for k, v in pairs(nodes) do
        if string.find(k, 'worldserver') then
            table.insert(servers, k)
        end
    end
    if 0 < #servers then
        return servers[ (self:getplayercount() % #servers) +1] --如果配置有多个则会根据配置的数量负载分配
    end
    return skynet.getenv("id") --默认获取本节点
  end
  ---调用场景服务的API
  function cworld:calltospace(pid, spacemailbox, func, ...)
    if spacemailbox.worldserver == skynet.getenv('id') then --区分是本节点内或集群的场景服务
      return  skynet.call(spacemailbox.spaceaddr, "lua", func, pid, table.unpack(params) )
    else
      return cluster.call(spacemailbox.worldserver, spacemailbox.spaceaddr, func, pid,table.unpack(params) ) --处于集群中的场景服务
    end
  end
  ```

  * `src/app/config/nodes.lua` 配置

    ```lua
    ...
    gameserver_lyh = {
            address = "127.0.0.1:1301",
            index = 102,
            -- worldserver = true, ?
        },
    ```

  * `src/app/game/world.lua` -> `createspace()`  -> `returen spacemailbox` 创建场景服务与信箱对象
    <a id="createSpacemailbox"></a>
  
    ```lua
    function cworld:createspace(spacedata)
      local spacemailbox = {
          worldserver =  self:getworldserver(),
          serverid = skynet.getenv("id"),
          spaceguid = spacedata.spaceguid, --全局唯一id
          spaceid = spacedata.spaceid, --场景id
          nodename = spacedata.nodename, --节点名
          spaceaddr = 0
      }
      return self:createspaceimp(spacedata, spacemailbox)
    end

    ---创建场景服务
    function cworld:createspaceimp(spacedata, spacemailbox)
        if spacemailbox.worldserver == skynet.getenv('id') then
            spacemailbox = skynet.call(".world", "lua", "createspace", spacemailbox, spacedata) --可以直接在当前节点创建场景服务
        else
            spacemailbox = gg.actor.cluster:call(spacemailbox.worldserver, ".game", "exec", "gg.world:createspaceimp", spacedata, spacemailbox) --调用集群的其他节点中的world服务来创建场景
        end
        if not spacemailbox then
            return
        end
        return  spacemailbox
    end
    ```

#### world 服务

* `src/app/world/world.lua`
  
  ```lua
  function cworld:createspace(spacemailbox, spacedata, args) --创建场景服务
    if not self.spaces[spacemailbox.spaceguid] then --创建新场景服务,并调用init函数
      local spaceaddr = skynet.newservice("app/world/space")
      local ret, data = self:safecall(spaceaddr, 'lua', 'init', spacemailbox, spacedata)
      ...
    end
  end
  ```

* `src/app/world/space.lua`

  ```lua
  ---服务初始化
  function space:init(spacemailbox, spacedata)
    ...
    spacemailbox.spaceaddr = ".space." .. spacedata.spaceguid --专属场景服务id 本地别名
    skynet.register(spacemailbox.spaceaddr, skynet.self()) --注册服务别名到节点中

    self.node = gg.class[spacedata.nodename].new(spacemailbox , spacedata) --"nodename":"cnodepersonal" 初始化 nodepersonal
    ...
    self:ontick() --
    return spacemailbox
  end

  function space:ontick()
      skynet.timeout(3, function () --30毫秒tick一次
          space:updatetimer()
          space:ontick()
      end
      )
  end

  function space:updatetimer()
      --TODO建一个队列处理
      gg.timer:update()
      self.timer:update()
      if self.node then
          self.node:update() --调用 src/app/world/node.lua中的update
      end
  end

  ---服务启动
  skynet.start(function ()
    skynet.dispatch("lua", xxx)
    ...
    space:tick() --间隔50秒判断一次是否要回收当前场景服务
    space:updatespaceinfo()
  end)

  ---玩家进入场景
  function space:enterspaceimp(entityid, spacemailbox, spacedata, celldata, inprogress, missiondata, reconnect )
    if self.node.__name == gg.commondefine.NODE_TYPE_PERSONAL and not reconnect then
      self:init(spacemailbox, spacedata) --如果非重连 则先初始化space
    end
    ...
    if not self.entities[entityid] then
        entity = self:loadentity(entityid, celldata) --加载实体数据
    end
    ...
    self.node:enter(entity, pos)
    if self.node.__name ~= gg.commondefine.NODE_TYPE_GLOBAL then --非全局node
        entity.attrs:syncdata() --同步数据
        self.node:syncenterspace(entity,spacedata,missiondata, reconnect) --同步进入场景
        entity.netcom:sendentityskilllist() --发送实体技能列表
        self:ping(entityid, {remotetime=0,returntime=0})
    end
  end
  ```

* `src/app/world/node.lua` <a id="node_sendtogameapp"></a>
  
  ```lua
  ---进入
  function node:enter(entity, pos, dir )
    ...
    self:addentity(entity) --添加实体数据
    self.map:enter(entity, pos, dir) --进入地图模块
    self:updateposition(entity.guid,pos,dir ) --更新位置
    entity:fireevent(commondefine.EVENT_ENTER_MAP, self) --触发进入地图事件
    entity:onenter(firstenter) -- ccharacter:onenter(first)
    return entity
  end
  ---调用 game/player/playerlogic:sendtoclient(cmd, guid, ...)
  function node:sendtoclient(entity, cmdname, ...)
    self:sendtogameapp(entity, "toclient", entity.attrs.guid, cmdname, ...) --下发到前端
  end
  function node:sendtoplayer(entity, cmdname, ...)
    self:sendtogameapp(entity, "toplayer", entity.attrs.guid, cmdname, entity.attrs.guid, ...) --调用 game/player/playerlogic.lua 对应函数
  end
  ---向game服务发送数据 对应到 src/app/common/actor/actor.lua 分发消息 类型为: world
  function node:sendtogameapp(entity, cmdname, ...)
      if entity.gameapp.serverid == skynet.getenv("id") then
          skynet.send(entity.gameapp.addr, 'lua', 'world' ,cmdname, ...)
      else
          cluster.send(entity.gameapp.serverid, entity.gameapp.addr, "world", cmdname, ...)
      end
  end
  ```

  * [actor.lua 分发消息 类型为: world](#actor_dispatch)
  * [场景邮箱 spacemailbox](#createSpacemailbox)
  * `nodepublic` : 共斗场景
  * `nodepersonal` : 个人场景
* `src/app/world/spacendoes/nodepersonal.lua` 个人场景

  ```lua
  ---继承自src/app/world/node.lua
  local cnodepersonal = class("cnodepersonal", gg.class.cnode)
  ---初始化函数
  function cnodepersonal:init(spacemailbox, spacedata )
    self.map = gg.class.cmapbase.new(spacedata.spaceid)
    gg.class.cnode.init(self, spacemailbox, spacedata) --world/node:init
    ...
  end
  ```

  `player` 类型对应的是:`src/app/world/unit/ccharacter.lua` 继承有:
    1. `aiunit`
    1. `unit`
    1. `entity`
    1. `event`
    1. `clientinterface`
* `src/app/world/spacendoes/nodepublic.lua` 共斗场景
* `src/app/world/map/mapbase.lua`  ?

### 驱动帧逻辑

* `src/app/world/node.lua` 场景帧驱动 <a id="fpsFrameUpdate"></a>

  ```lua
  ---30毫秒一帧 space:updatetimer()
  function node:update()
      self.timer:update()
      
      if self.fpsutility.FPSRATE <= self.fpsutility:interval() then --fpsutility.FPSRATE=33 判断上一帧的间隔
          local remain =   self:updateone()
          self.fpsutility:update(lutil.getms()) --更新fps间隔时间 world/utility/fpsutility:update
          --skynet.sleep(math.floor( remain/10))
      end
  end
  ---驱动一帧逻辑
  function node:updateone(  )
      self:updatebefore()
      for k, v in pairs(self.entities) do
          v:fireevent(commondefine.EVENT_UPDATE_TICK) --发送帧驱动事件
      end
      self:lmapupdateone()
      self:updateafter()
      for k, v in pairs(self.tickendfunclist) do
          v()
      end
      self.tickendfunclist = {}
      self:flushnet() --下发数据 调用:world/components/nettcomponet:flushall()
      return self.fpsutility:guessnettick(lutil.getms()) --计算下一帧间隔毫秒
  end
  ```

* `src/app/world/unit/ccharacter.lua` 注册网络组件

  ```lua
  function ccharacter:registercommponets()
    self:registercomponent('netcom', gg.class.cnettcomponet)
  end
  ```

* `src/app/world/components/nettcomponet.lua` [网络组件 用于数据发送](#nettcomponet)
  
  ```lua
  function cnettcomponet:flushall()
      self:flushsnapshot() --快照数据
      self:flush()  --其他数据
      self:flushrpc() --rpc同步数据
  end
  ```

### FPS

* `src/app/world/utility/fpsutility.lua` 记录每帧更新的时间间隔,用于判断是否进入下一帧
  
  ```lua
  function cfpsutility:update(current_tick)
    ...
  end
  ```

### 提交玩家操作指令

* `tools/proto/protobuf/battle.proto` 协议文件
  
  ```pb
  message Command_Move //战场中的移动
  {
    Vector3 direction = 1;
    int32 speed = 2; // float * 1000
    int32 dt = 3; // float * 1000 移动时间, 默认33
    Vector3 position = 4;
    
    bool justupdatepos = 5;//只更新位置不广播
    int32 type = 6;//移动类型默认0:普通移动 1：技能同步位移
  }

  message CommandData {
    optional Command_EntityAttack command_entityattack = 1;
    optional Command_Move command_move = 2;
    optional Command_EntityAttackBreak command_attackbreak = 3;
    optional Command_LoadingProgress command_loadingprogress = 4;
    optional Command_EntityAttackSkipSection command_attackskipsection = 5;
  }

  // @id=3103
  message C2GS_World_InputCommand
  {
    string name = 1;
    CommandData commanddata = 2;
    int64 remotetime = 3;//前端当前时间毫秒
    int64 returntime = 4; //
    int32 lastsn = 5;  // 当前用户提交的，经服务端确认的最后一条输入的 SN, 沒有就填-1
  }
  ```

* `src/app/world/handlers/battlehandler.lua`
  
  ```lua
  --[3103] = "C2GS_World_InputCommand" 输入协议
  function cbattlehandler:C2GS_World_InputCommand(player, args)
    ...
    if args.commanddata then
        self:parseinputcommand(player, args)
    end
    ...
  end
  --解析输入指令
  function cbattlehandler:parseinputcommand(player, args)
    local commanddata = args.commanddata
    player.netcom.lastsn = args.lastsn
    for k, v in pairs(commanddata) do
        local func = self[k]
        func(self, player, v)
    end
    ...
  end

  function cbattlehandler:command_move(player, args)
    ...
    --更新玩家位置和方向 world/controlers/charactercontroler 角色控制器
    player.controler:movelocation(
          gg.vector3({position.x, position.y, position.z}),
          gg.vector3({direction.x, direction.y, direction.z}),
          args.justupdatepos, 
          args.type
      )
    ...
  end
  ```

* `src/app/world/controlers/charactercontroler.lua`
  
  ```lua
  ---移动位置
  --@param justupdatepost 是否只更新位置不广播
  function ccharactercontroler:movelocation(position,direction, justupdatepos, moveType)
    ...
    if not self.owner:setposition(position) then --设置左边数据
            return
    end

    if not justupdatepos then
          self.owner.netcom:broadcastentitymove({ --广播位置更新数据
              direction = direction,
              targetposition = position + direction * math.floor(self.owner.ackcom.estimatertt / 1000),
              type = moveType
          })
    end
  end
  ```

### 同步玩家操作数据

* `tools/proto/protobuf/battle.proto` 协议文件
  
  ```pb
  message RPC_EntityMove
  {
    Vector3 direction = 2;
    Vector3 targetposition = 3;
    int32 type =4; //移动类型，move:0 普通移动，attack:1 技能移动
  }
  message RpcData
  {
    optional RPC_EntityMove rpc_entitymove = 1;
    optional RPC_EntityMoveBreak rpc_entitymovebreak = 2;
    xxx
  }

  message EntityRpcData {
    int32 id = 1;
    string name = 2;
    RpcData rpcdata= 4;
    int32 lastsn = 5;  // 当前用户提交的，经服务端确认的最后一条输入的 SN, 沒有就填-1
  }
  // @id=4101
  message GS2C_World_EntityRpc
  {
    repeated EntityRpcData rpcdatalist = 1;
    int64 remotetime = 2; //后端当前时间毫秒
    int64 returntime = 3; 
    int64 serverframe = 4;//当前后端帧数
  }
  ```

* `src/app/world/components/nettcomponet.lua` 角色网络组件 <a id="nettcomponet"></a>
  
  ```lua
  ---下发移动数据
  function cnettcomponet:sendentitymove(data)
      data.direction = data.direction and gg.mathutility:f2n(data.direction)
      data.targetposition = data.targetposition and gg.mathutility:f2n(data.targetposition)
      data.type = data.type or 0
      self:sendrpctoclient({name = 'rpc_entitymove', data = data})
  end
  ---广播移动数据
  function cnettcomponet:broadcastentitymove(data)
      data.direction = data.direction and gg.mathutility:f2n(data.direction)
      data.targetposition = data.targetposition and gg.mathutility:f2n(data.targetposition)
      data.type = data.type or 0
      self:broadcastrpctoall({name = 'rpc_entitymove', data = data})
  end
  ---将数据同步给client
  function cnettcomponet:sendrpctoclient(rpcdata) 
      rpcdata.id = self.owner.id
      self:sendtoclient('GS2C_World_EntityRpc', rpcdata)
  end
  ---将数据广播给场景中所以player
  function cnettcomponet:broadcastrpctoall(rpcdata)
    xxx
  end
  ---将数据广播给场景中所以player, 剔除self
  function cnettcomponet:broadcastrpctoothers(rpcdata)
    xxx
  end

  function cnettcomponet:sendtoclient(cmdname, ...)
    --rpc数据和snapshot快照数据将被直接下发
    if 'GS2C_World_EntityRpc' == cmdname then
        local rpcdata = self:newrpc(...)
        self:appendrpc(rpcdata) 
        return
    end
    if 'GS2C_World_EntitySnapshot' == cmdname then
        self:appendsnapshot(...)
        return
    end
    --其他协议数据将通过 cnettcomponet:flush() 每帧更新
    local params = table.pack(...)
    self:addnetfunc(function ()
      self.owner.node:sendtoclient(self.owner, cmdname, table.unpack(params)) --src/app/world/node:sendtoclient()
      end)
    ...
  end

  function cnettcomponet:appendrpc(rpcdata)
      table.insert(self.rpclist, rpcdata)
      self:flushrpc()
  end

  function cnettcomponet:flushrpc()
    ...
    self.owner.node:sendtoclient(self.owner, 'GS2C_World_EntityRpc', { 
        rpcdatalist = rpcdatalist,
        remotetime = gg.fpsutility:getservertime(),
        serverframe = self.owner.node:getcurrframecount()
    })
  end
  ```

  * `node:sendtoclient` [下发数据到client](#node_sendtogameapp)

## 玩家角色

* `src/app/world/unit/character.lua`
  
  ```lua
  function cnettcomponet:ctor()
      gg.class.ccomponent.ctor(self)
      self.netfuncs = {} --网络消息函数 sendXXX
      self.sync = false
      self.rpclist = {} --rcp调用
      self.snapshotlist = {} --快照
  end

  function ccharacter:init(celldata)
    ...
    self:setcontroler(gg.class.ccharactercontroler.new()) --角色控制器
  end
  ---注册组件
  function ccharacter:registercommponets()
    self:registercomponent('netcom', gg.class.cnettcomponet)
    self:registercomponent('skillcom', gg.class.cskillcomponent)
    self:registercomponent('bagcom', gg.class.cbagcomponent)
    self:registercomponent('agticom', gg.class.cagticomponent)
    self:registercomponent('equipcom', gg.class.cequipcomponent)
    xxx
  end
  ```

## rankor 排行榜服务

`src/app/world/services/rankor.lua`

## teamor 组队服务

`src/app/world/services/teamor.lua`

## gg > base > util 扩展工具包

分别包含:`string`,`table`,`functions`,`util`

### string 扩展

### table 扩展

### util uuid生成器

## 协议测试 client

脚本文件: `client/app/app.lua`

* 启动方式 `./client/3rd/lua/lua ./client/app/app.lua`

## 备忘

### 错误码响应 /src/app/common/http/answer.lua

### client 测试工程编译避坑

SVN 下载代码后不要直接使用 `make linux` 编译, 可能造成 `lua` 版本不正确, 正确的版本是`Lua 5.3.5`

具体操作方式如下:

```sh
# 删除第三方库
make delete3rd
# 检出第三方库 
make update3rd
# 编译
make linux

$ ./client/3rd/lua/lua -v #检查编译后的版本
$ Lua 5.3.5  Copyright (C) 1994-2018 Lua.org, PUC-Rio
```

### gm指令

* 热更模块: `sh gm.sh 0 hotfix app.game.client.login`  
  * 登陆服 http相关服务不能热更 `src/app/common/gg/hotfix.lua`

    ```lua
    gg.ignore_modules = {
        "gg%.service%.*",
        "gg%.like_skynet.*",
        "app%.game%.main",
        "app%.game%.httpd_main",
        "app%.scene%.main",
    }
    ```
