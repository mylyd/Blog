

### Android 接入Google Play 订阅以及一次性购买功能开发流程

**1.接入类型介绍**
		google支付方式有很多种方式，其中使用最多的是引入aidl开发，但是对于过于累赘的aidl接入模式  google官方推出了`Google Play 结算库`，下面主要讲解一下`Google Play 结算库`的使用流程以及避免一些不必要去踩踏的坑。

**2.接入具体流程**

  + [官方文档地址](https://developer.android.com/google/play/billing/billing_library_overview)

  + ```java
    implementation 'com.android.billingclient:billing:2.0.3'  //添加依赖
    ```
 1.第一步需要与google play 先连接起来

    ```java
    private BillingClient billingClient ;
    billingClient = BillingClient.newBuilder(this).enablePendingPurchases().setListener(this).build();
            if (billingClient == null){ //做非空判断
                return;
            }
            billingClient.startConnection(new BillingClientStateListener() {
                @Override
                public void onBillingSetupFinished(BillingResult billingResult) {
                        //处理链接成功后的逻辑，例如：进行订阅服务或者一次性购买的初始化
                }
                @Override
                public void onBillingServiceDisconnected() {
                 		//这里处理链接失败的逻辑，一般链接失败有常见的2种  一是，网络问题（国内需要翻墙）二是，手机上没						有安装Google Play ,处理逻辑自行把捏
                }
            });
    ```
2.创建你的订单，谷歌结算库不需要你的秘钥以及种种配置，只需要你填写进你的`商品id`,商品id在谷歌支付后台自行创建，这里不去讲解
    
    <*订阅或者一次性初始化*>
    
    ```java
    					//subs 订阅支付部分
                        SkuDetailsParams.Builder params = SkuDetailsParams.newBuilder();
                        List<String> ids = new ArrayList<>();
                        ids.add("xxxxxxxx"); //你创建的订阅商品id
                        params.setSkusList(ids).setType(BillingClient.SkuType.SUBS);//SUBS
                        billingClient.querySkuDetailsAsync(params.build(), new SkuDetailsResponseListener() {
                            @Override
                            public void onSkuDetailsResponse(BillingResult billingResult, List<SkuDetails> skuDetailsList) {
                                if (CollectionUtils.isEmpty(skuDetailsList)) {
                                    //获取订阅资源失败提示用户网络问题或者未安装GP
    								return;
                                }
                                XXXActivity.this.skuDetailsList = skuDetailsList;
    ```
    
    +  `SkuDetails`这个类中`getSubscriptionPeriod()` 方法会返回你订阅的周期信息 （`P1W`相当于一周，`P1M`相当于一个月，`P3M`相当于三个月，`P6M`相当于六个月，而`P1Y`相当于一年），这里周期性问题不需要去考虑，一般都是商品后台设置好的，你只用请求数据就可以了,但是需要开发去配合开发周期性逻辑
    
    + `querySkuDetailsAsync()`是谷歌提供封装好的执行网络查询并获取SKU详细信息且异步返回结果的一个回调方法；`List<SkuDetails>` 里面就是转载了你初始化好了的商品id
    
    + 一次性部分你只用将`BillingClient.SkuType.SUBS改成BillingClient.SkuType.INAPP`，当然你在List中填入的商品id也需要改成你一次性商品的id
    
    + 需要同时使用订阅跟一次性支付时，你需要在`onBillingSetupFinished()`中将上面的代码执行2次，下面看下
    
      `BillingClient`类`SkuType`部分源码，目前订阅库仅支持这2种付款分离开的调用方式，并没有让二者融合减少代码量，所以意味着上述查看到订阅部分初始化的内容你需要复制粘贴一次，并且在`onBillingSetupFinished()`中调用
    
	 ```java
       @Retention(SOURCE)
        public @interface SkuType {
          /** A type of SKU for in-app products. */
          String INAPP = "inapp";
          /** A type of SKU for subscriptions. */
          String SUBS = "subs";
        }
	 ```
 3.展示你的订单
     <*订阅或者一次性初始化展示*>
    
     ```java
       if (CollectionUtils.isEmpty(skuDetailsList)) {
                return;
            }
            if (billingClient.isFeatureSupported(BillingClient.FeatureType.SUBSCRIPTIONS).getResponseCode() != BillingClient.BillingResponseCode.OK) {
                //用户机型不支持GP支付
                return;
            }
            BillingFlowParams params = BillingFlowParams.newBuilder()
                    .setSkuDetails(skuDetailsList.get(0))
                    .build();
            BillingResult billingResult = billingClient.launchBillingFlow(this, params);
            Log.d(TAG, "sub: "+billingResult.getResponseCode());
     ```
    
    + `billingClient.isFeatureSupported()`方法主要是检测用户手机时候支持GP订阅支付功能,早些年期间GP推出订阅服务时手机很多版本是不支持订阅服务，这里需要去判断是否支持才能进行展示
    
    + 一次性支付功能同样如此只需要将传入的数据改变一下，当`billingResult.getResponseCode()`返回为0的时候代表展示成功
    
    + 下面是各种类型返回值（我目前只遇到了这2种返回情况，具体更多的看官网   [返回值及含义](https://developer.android.com/reference/com/android/billingclient/api/BillingClient.BillingResponse)）
    
      | 常熟值 |            含义            |
      | :----: | :------------------------: |
      |   0    |            成功            |
      |   7    | 物品已经拥有，未能成功购买 |
      
    + 注意一个点，一次性商品购买完成后做逻辑处理同时需要判断时候是消耗品，如果属于消耗品需要手动去消耗掉，不然后续一次性购买会失败也就是获得返回值为7，下面是消耗商品使用的代码（当然你需要先去获得单次内购是否是成功的，成功后你需要获得商品目前具体状态，判断时候是消耗品并且没有消耗）
    
    + `Purchase` 对象包含 [`isAcknowledged()`](https://developer.android.com/reference/com/android/billingclient/api/Purchase#isAcknowledged) 方法，该方法会指示购买交易是否已得到确认。此外，服务器端 API 包含 `Product.purchases.get()` 和 `Product.subscriptions.get()` 的确认布尔值。在确认购买交易之前，请使用这些方法来确定购买交易是否已得到确认。
    
    
    ```java
    void consumeAsync (ConsumeParams consumeParams, 
                  ConsumeResponseListener listener) //对于消耗性商品时调用
      void acknowledgePurchase (AcknowledgePurchaseParams params, 
                  AcknowledgePurchaseResponseListener listener)//对于非消耗性商品时调用
        //还可以使用服务器 API 中新增的 acknowledge() 方法
        boolean isAcknowledged ();
    ```
    具体实现调用
	
      ``` java
      billingClient.consumeAsync( //消耗单次内购次数
                     ConsumeParams.newBuilder().setPurchaseToken(purchases.get(0).getPurchaseToken()).build(), new ConsumeResponseListener() {
                     @Override
                       public void onConsumeResponse(BillingResult billingResult, String purchaseToken) {
                        if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK){
                                  Log.d(TAG, "onConsumeResponse: ");
                            }
                         }
                     });
      ```
      4.对购买结果进行处理
    
      ``` java
      @Override
          public void onPurchasesUpdated(BillingResult billingResult, @Nullable List<Purchase>  purchases) {
          }
      ```
    
      + setListener(this)上面初始化时会创建`onPurchasesUpdated`方法，具体逻辑以及需要的数据在Purchase类中都能调用到，在此处直接处理订阅完成后的逻辑，前提是你需要`billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK`判断时候执行成功，同时你也需要去`CollectionUtils.isEmpty(purchases)`确定`Purchases`不为空。
    

**3.查询订阅状态**

 具在查询这块遇到了一些坑人的点，用到查询无非是判定订阅状态，起初是觉得google在服务查询这一点应该是远程		异步加载查询，然后看了文档发现有[`queryPurchases()`](https://developer.android.com/reference/com/android/billingclient/api/BillingClient#queryPurchases(java.lang.String)) 方法（使用 Google Play 商店应用的缓存，而不发起网络请		求） [`queryPurchaseHistoryAsync()`](https://developer.android.com/reference/com/android/billingclient/api/BillingClient#queryPurchaseHistoryAsync(java.lang.String, com.android.billingclient.api.PurchaseHistoryResponseListener))方法（传入购买类型和 [`PurchaseHistoryResponseListener`](https://developer.android.com/reference/com/android/billingclient/api/PurchaseHistoryResponseListener) 以处理查询结果，发		起网络请求）

*queryPurchaseHistoryAsync（）的实现*

```java
billingClient.queryPurchaseHistoryAsync(SkuType.INAPP,  new PurchaseHistoryResponseListener() {   
    @Override    
    public void onPurchaseHistoryResponse(BillingResult billingResult,  List<Purchase> purchasesList) {  
        if (billingResult.getResponseCode() == BillingResponse.OK && purchasesList != null) {      
            for (Purchase purchase : purchasesList) {       
                  // Process the result.       
            }       
        }   
    }
});
```

+ 具体查询订阅状态主要是通过使用 [`Purchase.getPurchaseState()`](https://developer.android.com/reference/com/android/billingclient/api/Purchase#getpurchasestate) 方法确定购买状态是 `PURCHASED` 还是 `PENDING`。注意，只有在状态为 `PURCHASED` 时，才能授权通过执行以下操作来检查状态更改：

  1. 启动应用时，调用 [`BillingClient.queryPurchases()`](https://developer.android.com/reference/com/android/billingclient/api/BillingClient#querypurchases) 以检索与用户关联的非消耗型商品的列表，然后对返回的每个 `Purchase` 对象调用 `getPurchaseState()`。
  2. 实现 [`onPurchasesUpdated()`](https://developer.android.com/reference/com/android/billingclient/api/PurchasesUpdatedListener#onpurchasesupdated) 方法以响应对 `Purchase` 对象的更改。

+ 在其他类中查询订阅状态时，需要重新`new BillingClient`  并且实现`onPurchasesUpdated()`方法，然后调用`queryPurchases()`, 示例

  *应用queryPurchases查询订阅状态*
  
  ```java
  public static boolean purchasesResult(BillingClient billingClient) {   
      Purchase.PurchasesResult purchasesResult = billingClient.queryPurchases(BillingClient.SkuType.SUBS);         
      List<Purchase> purchases = purchasesResult.getPurchasesList();   
      if (CollectionUtils.isEmpty(purchases)) {
          return false;   
      }   
      boolean isStatus = false;
      if (purchases.get(0).getPurchaseState() == Purchase.PurchaseState.PURCHASED) {    
        		isStatus = true;
      } 
      return isStatus;
  }
  ```
  

