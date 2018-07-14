# WebSocket-stomp
WebSocket-stomp的web客服系统实现

controller:

    @Controller
    public class WsController {
    @Autowired
    SimpMessagingTemplate messagingTemplate;

    @MessageMapping("/chat1")
    @SendToUser("/message")
    public void handleChat(@RequestBody WsRequest wsRequest) {
        String username=wsRequest.getUsername();
        String msg=wsRequest.getMsg();
        System.out.println("-------------"+username+"=========="+msg);
        messagingTemplate.convertAndSendToUser(username, "/message", msg);
    }

    @MessageMapping("/chat2")
    @SendTo("/topic/all")
    public void sendToAll(@RequestBody WsRequest wsRequest){
        String msg=wsRequest.getMsg();
        System.out.println("-------------"+msg);
        messagingTemplate.convertAndSend("/topic/all",msg);
    }

    }
    
配置类
    
    @Configuration
    @EnableWebSocketMessageBroker
    public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry stompEndpointRegistry) {
        stompEndpointRegistry.addEndpoint("/endpointChat")
                .setAllowedOrigins("*")
                .withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/queue","/topic");
        registry.setUserDestinationPrefix("/queue");
    }
    }
    
web界面
            <!DOCTYPE html>

        <html xmlns:th="http://www.thymeleaf.org">
        <meta charset="UTF-8"/>
        <head>
            <title>Home</title>
            <script src="https://cdn.bootcss.com/sockjs-client/1.1.4/sockjs.min.js"></script>
            <script src="https://cdn.bootcss.com/stomp.js/2.3.3/stomp.min.js"></script>
            <script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
        </head>
        <body>
        <p>
           客服
        </p>
        username:<input type="text" id="username" value="aaa">
        <button id="botton">建立连接</button>
        <form id="wiselyForm">
            <textarea rows="4" cols="60" name="text"></textarea>
            <input type="submit"/>
            <input type="button" id="sub" value="all">
        </form>

        <script th:inline="javascript">
          var  sock = new SockJS("/endpointChat");
          var  stomp = Stomp.over(sock);
          var username;
            $('#wiselyForm').submit(function (e) {
                e.preventDefault();
                var text = $('#wiselyForm').find('textarea[name="text"]').val();
                sendSpittle(username,text);
                //对单独用户发送消息
            });
            $("#sub").click(function () {
                var text = $('#wiselyForm').find('textarea[name="text"]').val();
                sendSpittleAll(text);
                //对所有用户发送消息
            })
            

            $("#botton").click(function () {
                    console.log("connecting");
                    username=$("#username").val();
                getconnect(stomp)
            });
            function getconnect (stomp) {
            //获取连接
                sock = new SockJS("/endpointChat");
                stomp = Stomp.over(sock);
                stomp.connect({}, function (frame) {
                    stomp.subscribe("/queue/" + username+ "/message", handleNotification);
                    //订阅此用户的消息
                    stomp.subscribe("/topic/all", handleNotificationAll);
                    //订阅所有消息
                });
            }

            function handleNotification(message) {
                $('#output').append("<b>Received: " + message.body + "</b><br/>")
            }
            function handleNotificationAll(message) {
                $('#output').append("<b>ReceivedFromAll: " + message.body + "</b><br/>")
            }

            function sendSpittle(username,text) {
                var body = {
                    "username": username,
                    "msg": text

                };
                // stomp.send("/chatl", {}, JSON.stringify({ 'guest': text }));//3
                stomp.send("/chat1", {}, JSON.stringify(body));//3
            }
            function sendSpittleAll(text) {
                var body = {
                    "msg": text

                }
                // stomp.send("/chatl", {}, JSON.stringify({ 'guest': text }));//3
                stomp.send("/chat2", {}, JSON.stringify(body));//3
            }

            $('#stop').click(function () {
                sock.close()
            });
        </script>

        <div id="output"></div>
        </body>
        </html>
