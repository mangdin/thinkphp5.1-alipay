# thinkphp5.1_alipay
thinkphp5.1 支付宝

将根目录的aliyun.php拷贝到config目录

公共函数：
    <pre>    
    /**
     * 跳向支付宝付款
     * @param  array $order 订单数据 必须包含 out_trade_no(订单号)、price(订单金额)、subject(商品名称标题)
     */
    function alipay($order){
        // 获取配置
        $config=\think\facade\Config::get('alipay.');
        $data=array(
            "_input_charset" => 'utf-8', // 编码格式
            "logistics_fee" => "0.00", // 物流费用
            "logistics_payment" => "SELLER_PAY", // 物流支付方式SELLER_PAY（卖家承担运费）、BUYER_PAY（买家承担运费）
            "logistics_type" => "EXPRESS", // 物流类型EXPRESS（快递）、POST（平邮）、EMS（EMS）
            "notify_url" => 'http://'.$_SERVER['HTTP_HOST'].'/index/alipay/alipay_notify', // 异步接收支付状态通知的链接
            "out_trade_no" => $order['out_trade_no'], // 订单号
            "partner" => $config['partner'], // partner 从支付宝商户版个人中心获取
            "payment_type" => "1", // 支付类型对应请求时的 payment_type 参数,原样返回。固定设置为1即可
            "price" => $order['price'], // 订单价格单位为元
            // "price" => 0.01, // // 调价用于测试
            "quantity" => "1", // price、quantity 能代替 total_fee。 即存在 total_fee,就不能存在 price 和 quantity;存在 price、quantity, 就不能存在 total_fee。 （没绕明白；好吧；那无视这个参数即可）
            "receive_address" => '1', // 收货人地址 即时到账方式无视此参数即可
            "receive_mobile" => '1', // 收货人手机号码 即时到账方式无视即可
            "receive_name" => '1', // 收货人姓名 即时到账方式无视即可
            "receive_zip" => '1', // 收货人邮编 即时到账方式无视即可
            "return_url" => 'http://'.$_SERVER['HTTP_HOST'].'/index/alipay/alipay_return', // 页面跳转 同步通知 页面路径 支付宝处理完请求后,当前页面自 动跳转到商户网站里指定页面的 http 路径。
            "seller_email" => $config['seller_email'], // email 从支付宝商户版个人中心获取
            "service" => "create_direct_pay_by_user", // 接口名称 固定设置为create_direct_pay_by_user
            "show_url" => $order['show_url'], // 商品展示网址,收银台页面上,商品展示的超链接。
            "subject" => $order['subject'] // 商品名称商品的标题/交易标题/订单标 题/订单关键字等
        );
        $alipay = new \mangdin\alipay\AlipaySubmit($config);
        $new=$alipay->buildRequestPara($data);
        $go_pay=$alipay->buildRequestForm($new, 'get','支付');
        echo $go_pay;
    }
    </pre>
    
    调用支付控制器代码:
    <pre>
    /**
     * 支付宝PC扫码支付
     * @param $orderid  订单ID
     */
    public function alipay($orderid){
        $data=array(
            'out_trade_no'  =>  $subject['sn'],             //订单编号
            'price'         =>  $price,                     //订单价格
            'subject'       =>  $subject['goods']['title'],        //商品名称商品的标题/交易标题/订单标 题/订单关键字等
            "show_url"      =>  'http://'.$_SERVER['HTTP_HOST']    // 商品展示网址,收银台页面上,商品展示的超链接（支付界面标题可以跳转链接）。
        );
        alipay($data);
    }
    </pre>
    
    支付回调控制器代码：
    <pre>
    /**
     * return_url接收页面
     */
    public function alipay_return(){
        // 引入支付宝
        $config=\think\facade\Config::get('alipay.');
        $notify=new \mangdin\alipay\AlipayNotify($config);
        // 验证支付数据
        $status=$notify->verifyReturn();
        if($status){
            // 下面写验证通过的逻辑 比如说更改订单状态等等 $_GET['out_trade_no'] 为订单号；
           
            $this->success('支付成功',url('index/user/order'));
        }else{
            $this->success('支付失败',url('index/user/order'));
        }
    }

    /**
     * notify_url接收页面
     */
    public function alipay_notify(){
        // 引入支付宝
        $config=\think\facade\Config::get('alipay.');
        $alipayNotify = new \mangdin\alipay\AlipayNotify($config);
        // 验证支付数据
        $verify_result = $alipayNotify->verifyNotify();
        if($verify_result) {
            // 下面写验证通过的逻辑 比如说更改订单状态等等 $_POST['out_trade_no'] 为订单号；
            
            echo "success";
        }else {
            echo "fail";
        }
    }
    </pre>
