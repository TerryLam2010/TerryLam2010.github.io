# 熊掌号TP开通流程以及后续api调用

一、TP开通

1. 通过百度企业认证，填写相关信息

2. 编写接口

   只需返回success字符串给百度即可

   (1). 接受熊掌号授权信息、接受ticket

   contentType为application/json

   ps:不知道为什么第一次发给我是 application/x-www-form-urlencoded 

   ```
    /**
        * 接受熊掌号授权信息、接受ticket
        * @param request
        * @return success
        * @author TerryLam
        *
        */
       @RequestMapping(value = "/authorization",method = RequestMethod.POST)
       @ResponseBody
       @ApiOperation("接受熊掌号授权信息、接受ticket")
       public String authorization(HttpServletRequest request,
                                   @ApiParam(value = "ticket 和 授权信息")
                                   @RequestBody String requestBody
                                   ) throws XzhException{
           // System.out.println("authorization:"+request.getContentType());
           String signature = request.getParameter("signature");
           String timestamp = request.getParameter("timestamp");
           String encrypt_type = request.getParameter("encrypt_type");
           String msg_signature = request.getParameter("msg_signature");
           String nonce = request.getParameter("nonce");
   
           AesEncryptUtil crypt = new AesEncryptUtil(XZHConstant.TOKEN, XZHConstant.ENCODING_AES_KEY, XZHConstant.CLIENT_ID);
   
           // 使用JSON对象解析jsonString
           JSONObject jsonRequestBody = JSON.parseObject(requestBody);
           // 从json中获取加密信息
           String encrypt = jsonRequestBody.getString("Encrypt");
           // 解密
           String decrypt = crypt.decrypt(encrypt);
           // 使用JSON对象解析解密jsonString
           JSONObject jsonAuthorization = JSON.parseObject(decrypt);
           // 获取类型
           String infoType = jsonAuthorization.getString("InfoType");
   
           // TODO save to database
           if("tp_verify_ticket".equals(infoType)){
               // 推送ticket
               Ticket ticket = JSON.parseObject(decrypt, Ticket.class);
               ticketService.saveTicket(ticket);
           }else{
               // 授权信息推送
               AuthorizationEvent authorizationEvent = JSON.parseObject(decrypt, AuthorizationEvent.class);
               authorizationEventService.saveEvent(authorizationEvent);
           }
   
           return "success";
       }
   ```

   (2). 接受熊掌号事件推送

   ```
   /**
        * 接受熊掌号事件推送
        * @param request
        * @return success
        * @author TerryLam
        */
       @RequestMapping(value = "/event",method = RequestMethod.POST)
       @ResponseBody
       @ApiOperation("接受熊掌号事件推送")
       public String event(HttpServletRequest request,
                           @ApiParam( value = "事件消息请求体", required = true )
                           @RequestBody String requestBody) throws XzhException {
          // System.out.println("event:"+request.getContentType());
           String signature = request.getParameter("signature");
           String timestamp = request.getParameter("timestamp");
           String encrypt_type = request.getParameter("encrypt_type");
           String msg_signature = request.getParameter("msg_signature");
           String nonce = request.getParameter("nonce");
   
           // 使用JSON对象解析jsonString
           JSONObject jsonRequestBody = JSON.parseObject(requestBody);
           String encrypt = jsonRequestBody.getString("Encrypt");
   
           AesEncryptUtil crypt = new AesEncryptUtil(XZHConstant.TOKEN, XZHConstant.ENCODING_AES_KEY, XZHConstant.CLIENT_ID);
           String decrypt = crypt.decryptJsonPassiveMsg(msg_signature, timestamp, nonce, encrypt);
   
           MsgEvent msgEvent = JSON.parseObject(decrypt, MsgEvent.class);
           // TODO save to database
           msgEventService.saveMsgEvent(msgEvent);
           return "success";
       }
   ```

   注： 调用的api使用百度提供的xzh-sdk

   (3). 保存后全网发布即可

   

二、熊掌号授权