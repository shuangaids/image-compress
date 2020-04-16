# image-compress

复制粘贴图片，因为图片太大，上传速度太慢，因此整改，使用了canvas对进行上传的图片进行压缩，代码如下

  fun = (e) => {
  
    var blob;
    
    //获取clipboardData对象
    
    var data = e.clipboardData || window.clipboardData;
    
    //获取图片内容
    
    blob = data.items[0].getAsFile();
    
    //判断是不是图片，最好通过文件类型判断
    
    var isImg = (blob && 1) || -1;
    
    var reader = new FileReader();
    
    if (isImg >= 0) {
    
      //将文件读取为 DataURL
      
      reader.readAsDataURL(blob);
      
    }
    
    reader.onloadend = async (event) => {
    
      //获取base64流
      
      var base64_str = reader.result;
      
      var img = new Image();
      
      img.onload = () => {
      
        var canvas = document.createElement('canvas');
        
        var ctx = canvas.getContext('2d');
        
        canvas.width = img.width;
        
        canvas.height = img.height;
        
        ctx.drawImage(img, 0, 0);
        
        //压缩图片
        
        var newDataURL = canvas.toDataURL('image/jpeg', 0.9);
        
        this.setState({
        
          radioChangeimg: newDataURL,
          
        });
        
        canvas = null;
        
      };
      
      img.src = base64_str;     
      
    };
    
  };
  
  //7牛云上传图片

  router.post('/invoice/upload1', upload.any(), (req, res) => {
  
    let client = qn.create(qiniconfig);
    
    const params = req.body;
    
    let mac = new qiniu.auth.digest.Mac(qiniconfig.accessKey, qiniconfig.secretKey);
    
    let configs = new qiniu.conf.Config();
    
    let bucketManger = new qiniu.rs.BucketManager(mac, configs);
    
    var bitmap = new Buffer(params.img, 'base64');
    
    let key = `InputInvoice_${moment(Date.now())
    
      .format('YYYYMMDDHHmmss')}${moment()
      
      .millisecond()}.png`;
     
    client.upload(bitmap, {
    
      key
      
    }, function (err, result) {
    
      if (err) {
      
        res.status(200)
        
          .send({
          
            subCode: 1,
            
            msg: '上传失败'
            
          });
          
      } else {
      
        result.picRealurl = bucketManger.privateDownloadUrl(qiniconfig.origin, result.key, qiniconfig.timeout);
        
        res.status(200)
        
          .send({
          
            subCode: 0,
            
            data: result
            
          });
          
      }
      
    });
    
  });

