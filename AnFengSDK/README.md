# AnFengSDK

## 订阅功能


- **订阅商品id**

    com.subscription.rollergo.id1

- **订阅方法**  
  
  1.**订阅调用**

  ```java
  //cocos里对应调用android下的订阅方法,其中app对应当前Activity的实例，hasSubscription为AnFengSDK.showADRequest回调方法中已赋值的
    public static void subscript(){
        app.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                if(app.hasSubscription){
                    Toast.makeText(app, "当前已经订阅", Toast.LENGTH_SHORT).show();
                } else {
                    app.payToSubscription();
                }
            }
        });
    }  

    //实际订阅功能的对应方法，自定义的payType、SUBSCRIPTION：String，分别表示当点调用的订阅类型，自定义的SUBSCRIPTION_ID：String，表示对应订阅商品的id
    private void payToSubscription() {
        payType = SUBSCRIPTION;
        String orderId = String.valueOf(System.currentTimeMillis());
        OrderInfo info = new OrderInfo();
        info.setOrderId(orderId);
        info.setProductId(SUBSCRIPTION_ID);//内购商品ID，由运营提供
        info.setPayAmount("0.01");//价格
        info.setGoodsDesc("订阅商品0.01元");//描述
        info.setGoodsName("订阅商品");//详情
        AnFengSDK.pay(app, info);
    }
  ```

  2.**订阅相关回调**  

    2.1 **初始化时获取已订阅商品列表，处理对应订阅商品逻辑**

    ```java
        AnFengSDK.showADRequest(this, new RequestCallback<List<CpProductInfo>>() {
            @Override
            public void beginOnNetWork() {}

            @Override
            public void failedOnError(int errCode, String errMsg) {
                Log.e("failedOnError", errMsg);
            }
            @Override
            public void succeedOnResult(List<CpProductInfo> cpProductInfos) {
                //网络请求拉取已购买的商品列表，列表中存在某商品即代表已经购买
                //筛选List<CpProductInfo>中存在的计费点id，存在即代表已成功购买，不存在就代表没有付费购买
                if (cpProductInfos != null && cpProductInfos.size() > 0) {
                    for (int i = 0; i < cpProductInfos.size(); i++) {
                        String product_id=cpProductInfos.get(i).getCp_product_id();
                        //eg:订阅google.subscription._test.id.1(运营提供),如果存在了这个product_id表明已购买
                        if (SUBSCRIPTION_ID.equals(product_id)) {
                            Toast.makeText(app, "当前已经订阅", Toast.LENGTH_SHORT).show();
                            hasSubscription = true;
                        } else {
                            Toast.makeText(app, "当前没有订阅", Toast.LENGTH_SHORT).show();
                        }
                    }
                } else {
                    Toast.makeText(app, "当前没有订阅", Toast.LENGTH_SHORT).show();
                    Toast.makeText(app, "订阅2商品也没有订阅", Toast.LENGTH_SHORT).show();
                }
            }
  ```

  2.2 **订阅成功后，处理对应订阅商品逻辑**

  ```java
  AnFengSDK.sdkInit(app, new AnFengSDKListener() {...
   @Override
        public void onPaySuccess(String sdkOrderId) {
            LogUtil.e(TAG,"支付结果回传5");
            Toast.makeText(app, "支付提示：" + "订单  " + sdkOrderId + "  已支付成功", Toast.LENGTH_SHORT).show();
            //此处处理你的业务逻辑即可
            if (SUBSCRIPTION.equals(payType)){
                hasSubscription=true;//表示已经订阅成功，刷新标记状态为true
            }
        }

        @Override
        public void onPayFailure(String sdkOrderId) {
            LogUtil.e(TAG,"支付结果回传6");
            Toast.makeText(app, "支付失败", Toast.LENGTH_SHORT).show();

        }

        @Override
        public void onPayCancel() {
            LogUtil.e(TAG,"支付结果回传10");
            Toast.makeText(app, "放弃支付", Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onPayUnderway(String sdkOrderId) {
            Toast.makeText(app, "支付进行中", Toast.LENGTH_SHORT).show();
        }
  ```

