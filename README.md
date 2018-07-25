# thinkphp5.1_alipay
thinkphp5.1 支付宝

将根目录的aliyun_dysms.php拷贝到config目录

示例代码：
    /**
     * 阿里云 大于短信 发送接口
     * @param $mobile 手机号码
     * @param $signname 签名
     * @param $templatecode  短信模板ID
     * @param array $templateparam  短信模板 传入参数
     * @return \think\response\Json
     */
    public function sendsms($mobile,$signname,$templatecode,$templateparam=array()){
        $templateparam = array(
            "code" => rand(100000,999999)
        );
        $sendsms = new \mangdin\alidysms\sendSms();
        $msg = json_decode($sendsms->sendsms($mobile, $signname, $templatecode, $templateparam),true);
        if ($msg['Code'] == 'OK'){
            Session::set('smscode',$templateparam['code']);
            SmslogModel::create(array(
                'uid'=>  0,
                'mobile' => $mobile,
                'content' => $templateparam['code'],
                'addtime' => time()
            ));
            return json(['code'=>200,'icon'=>6,'msg'=>'发送成功']);
        }else{
            return json(['code'=>500,'icon'=>5,'msg'=>$msg['Message']]);
        }
    }
