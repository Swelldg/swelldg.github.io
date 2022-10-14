# PHP后端生成二维码

## 官方地址
### [endroid/qr-code](https://github.com/endroid/qr-code)
### [PHP-GD extension](https://www.php.net/manual/en/book.image.php)

## 安装
```composer require endroid/qr-code```

## 基本使用举例
### 后端
```php
use Endroid\QrCode\Builder\Builder;
use Endroid\QrCode\Color\Color;
use Endroid\QrCode\Encoding\Encoding;
use Endroid\QrCode\ErrorCorrectionLevel\ErrorCorrectionLevelLow;
use Endroid\QrCode\QrCode;
use Endroid\QrCode\Label\Label;
use Endroid\QrCode\Logo\Logo;
use Endroid\QrCode\RoundBlockSizeMode\RoundBlockSizeModeMargin;
use Endroid\QrCode\Writer\PngWriter;
use Endroid\QrCode\ErrorCorrectionLevel;
use Endroid\QrCode\ErrorCorrectionLevel\ErrorCorrectionLevelHigh;
use Endroid\QrCode\Label\Alignment\LabelAlignmentCenter;
use Endroid\QrCode\Label\Font\NotoSans;

$result = Builder::create()
    ->writer(new PngWriter())
    ->writerOptions([])
    ->data('https://yourwebsite.com')   //输入二维码跳转的网址，简单输入字符内容也可以
    ->encoding(new Encoding('UTF-8'))
    ->errorCorrectionLevel(new ErrorCorrectionLevelHigh())
    ->size(280)  //调整二维码图片大小
    ->margin(0)  //调整二维码图片外边距
    ->labelText('This is the label') //设置二维码下方label
    ->labelFont(new NotoSans(20)) 
    ->labelAlignment(new LabelAlignmentCenter())
    ->roundBlockSizeMode(new RoundBlockSizeModeMargin())
    ->build();
echo $result; //调试窗口查看生成好的二维码
$dataUri = $result->getDataUri(); //获取生成二维码的data URI,方便前端展示
return json_encode($dataUri);  //转化为json编码的string
```
### 前端
通过img标签，将获取到的json编码的dataUri传入src，展示二维码图片  
`<img src={dataUri}/>` 

## 使用PHP-GD extension添加中间logo
使用到的gd库中的函数都可以在官方文档中查询  
[PHP-GD](https://www.php.net/manual/en/book.image.php)
### 后端
```php
use Endroid\QrCode\Builder\Builder;
use Endroid\QrCode\Color\Color;
use Endroid\QrCode\Encoding\Encoding;
use Endroid\QrCode\ErrorCorrectionLevel\ErrorCorrectionLevelLow;
use Endroid\QrCode\QrCode;
use Endroid\QrCode\Label\Label;
use Endroid\QrCode\Logo\Logo;
use Endroid\QrCode\RoundBlockSizeMode\RoundBlockSizeModeMargin;
use Endroid\QrCode\Writer\PngWriter;
use Endroid\QrCode\ErrorCorrectionLevel;
use Endroid\QrCode\ErrorCorrectionLevel\ErrorCorrectionLevelHigh;
use Endroid\QrCode\Label\Alignment\LabelAlignmentCenter;
use Endroid\QrCode\Label\Font\NotoSans;
$result = Builder::create()
    ->writer(new PngWriter())
    ->writerOptions([])
    ->data('https://yourwebsite.com)
    ->encoding(new Encoding('UTF-8'))
    ->errorCorrectionLevel(new ErrorCorrectionLevelHigh())
    ->size(280)
    ->margin(0)
    ->roundBlockSizeMode(new RoundBlockSizeModeMargin())
    ->build();
header('Content-Type: '.$result->getMimeType());
$dataUri = $result->getDataUri();
$qrcode_img = imagecreatefrompng($dataUri);  //调用gd库，从png图片创建新图片(imagecreatefrompng)
$middle_img = imagecreatefromjpeg($middle_img_url);  //调用gd库，从jpeg创建新图片(imagecreatefromjpeg)
list($width, $height) = getimagesize($middle_img);  //调用getimagesize获取中间图片的原始宽高
$middle_img_resize = imagecreatetruecolor(50, 50);  //调用imagecreatetruecolor，创建新的大小为50*50的真彩色图片
$color = imagecolorallocate($middle_img_resize, 255, 255, 255);  //将新图片的背景设置为透明
imagefill($middle_img_resize, 0, 0, $color);
$source = $middle_img;
imagecopyresampled($zhuanlan_img_resize, $source, 0, 0, 0, 0, 50, 50, $width, $height);  //调用imagecopyresampled将原本的中间图片调整为大小为50*50的缩小图片(imagecopyresized会极大程度降低图片清晰度，不建议使用)
imagecopymerge($qrcode_img,$zhuanlan_img_resize,115,115,0,0,50,50,100);  //调用imagecopymerge将原始qrcode图片与中间图片拼接
ob_start ();
imagepng ($qrcode_img);
$image_data = ob_get_contents ();
ob_end_clean ();
$qrcode_img_uri = base64_encode($image_data);  //使用base64转码
return $qrcode_img_uri;    
```
### 前端
`<img src={"data:image/png;base64,"+qrcode_img_uri}/>`


