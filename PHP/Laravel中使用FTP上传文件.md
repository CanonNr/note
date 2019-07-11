## Laravel中使用FTP上传文件

Laravel中内置的FTP上传功能还是很好用的!

## 开始

- 第一步：设置配置文件

  

  ```php
  # config/filesystems.php
  'disks' => [
  	...
      'ftp' => [
              'driver'   => 'ftp',
              'host'     => '192.168.1.193',
              'username' => 'uenit',
              'password' => 'uenit',
              'port'     => 21,
              // 'root'     => '',
              // 'passive'  => true,
              // 'ssl'      => true,
              // 'timeout'  => 30,
          ],
  ]
  ```



- 第二步：直接文件上传就好了

  ```php
  ...
      
  if(!empty($request->file())){
     $file = $request->file('file');
     //题解
     if(!empty($writeup)){
         $fileExt = $file->getClientOriginalExtension();
         $realPath = $file->getRealPath();
         $filename = date('YmdHis') . uniqid() . '.' . $fileExt;
         $result = Storage::disk('ftp')->put($filename,file_get_contents($realPath));
         if($result){
             $model->file = $filename;
         }
     }
  }
  $model->save();
  ```

  

