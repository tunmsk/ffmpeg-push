# ffmpeg-push

    用ffmpeg命令推流库
  
# 安装

```cmd
composer require bping/ffmpeg-push
```

# 使用

### 首先

  * 安装[ffmpeg](http://ffmpeg.org/download.html)，必须同时包含`ffprobe`命令
  * 配置可执行文件目录到环境变量PATH中
    
### 推流

```php
require __DIR__ . '/trunk/vendor/autoload.php';

use FFMpegPush\PushFormat;
use FFMpegPush\PushInput;
use FFMpegPush\PushOutput;
use FFMpegPush\PushVideo;

 ///** @var  $ffprobe FFProbeCommand */
 //$ffprobe = FFProbeCommand::create();
 //var_dump($ffprobe->format('test.mp4'));

// 推流
// ffmpeg -re  -i  \"test/test.mp4\" -c:v copy -c:a copy -f flv rtmp://pili-publish.heliwebs.com
 $pushUrl = 'rtmp://pili-publish.heliwebs.com';
 $pushCmd = PushVideo::create();
 // 监听推流进度
 $pushCmd->onProgress(function ($percent, $remaining, $rate) {
 //    var_dump(func_get_args());
     echo "progress:$percent% remaining:$remaining(s) rate:$rate(kb/s)\n";
 });
 
 $pushCmd->setInput(
     PushInput::create()
         ->setStartTime(0)
         ->setInputVideo('res/test.mp4')
 )
     ->setFormat(
         PushFormat::create()
             ->setVideoCodec(PushFormat::CODE_V_COPY)
     )
     ->setOutput(
         PushOutput::create()
             ->setPushUrl($pushUrl)
     );
 
 echo $pushCmd->getCommandLine();
 
 // 开始推流
 $pushCmd->push();
 
 echo $pushCmd->getErrorOutput();
 echo "\n";
 echo "Exit Code: " . $pushCmd->getExitCode();   
        
// 停止推流，需要异步调用 
// $pushCmd->stop();           
```
### 结果 `PushInfo`

```php
//是否成功
  $pushinfo->isSuccessful()    
//输出
  $pushinfo->getOutput()    
//错误输出
  $pushinfo->getErrOutput()    
//执行返回码
  $pushinfo->getExitCode()
//目前推流时间，可以用中途断流重推起点时间
  $pushinfo->getCurrentTime()
//更多请看 PushInfo类  
```

### 输入 `PushInput`

```php
  PushInput::create()
  ->setStartTime(10)
  ->setInputVideo('test/test.mp4')
```

### 转码 `PushFormat`

```php
        PushFormat::create()
            ->setVideoCodec("x264")
            ->setAudioCodec('copy')
            ->setAudioKiloBitrate(125)
            ->setVideoKiloBitrate(500)
            ->setAdditionalParamaters(
                array(
                    '--preset',
                    'ultrafast',
                    ' --tune',
                    'zerolatency',
                )
            );
```

### 输出 `PushOutput`

```php
 PushOutput::create()->setPushUrl($pushUrl)
```

### 获取视频文件信息

```php
///** @var  $ffprobe FFProbeCommand */
$ffprobe = FFProbeCommand::create();
var_dump($ffprobe->format('test/test.mp4'));
var_dump($ffprobe->stream('test/test.mp4'));
```

# 配置

### `ffmpeg.binaries`:

> ffmpeg命令名称或者路径。如果想使用简单名称，记得把ffmpeg加入环境变量`PATH` 中。 默认值：ffmpeg

### `ffprobe.binaries`:

> ffprobe命令名称或者路径。如果想使用简单名称，记得把ffprobe加入环境变量`PATH` 中。 默认值：ffprobe

### `timeout`:

> 命令执行的超时时长，单位（s）.考虑到推流时长一般较长，所以默认值为一天


```php
$pushCmd = PushVideo::create(Configuration::create(
    array(
    'ffmpeg.binaries'=>array('ffmpeg'),
    'ffprobe.binaries'=>array('ffprobe'),
    'timeout'=>10800,
    )
));
```


# 主要依赖

* [Symfony/Process](https://github.com/symfony/symfony/tree/master/src/Symfony/Component/Process)
