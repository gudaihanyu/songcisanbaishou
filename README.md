```php
public function handle()
{
    $idx = 0;
    $dir = '/Users/mwh/Downloads/af/OEBPS/';
    $html = str_get_html(file_get_contents($dir . 'text00000.html'));

    $authorList = $this->getAuthor($dir . 'text00288.html');

    $preAuthor = '';
    $globleStr = '';

    $eles = $html->find('p');
    foreach($eles as $element){

        $pClazz = $element->getAttribute('class');

        $a = $element->find("a", 0);
        $txt = $a->plaintext;
        $href = $a->href;

        if($pClazz == 'mulu1') {
            $ar = explode('　', $txt);
            if(Str::length($ar[0]) == 1) {
                $author = $ar[0] . $ar[1];
            } else {
                $author = $ar[0];
            }

            if($preAuthor != '' && $preAuthor != $author) {
                file_put_contents('/Users/mwh/sphinx/songci/source/' . $idx++ . '.rst', $globleStr);
                $globleStr = '';
            }

            $authorStr = $author == '无名氏'? '' : array_shift($authorList);
            $this->info($author);
            $this->info($authorStr . PHP_EOL);
            $globleStr .= $author . PHP_EOL . '=========================' . PHP_EOL . PHP_EOL . $authorStr . PHP_EOL . PHP_EOL . PHP_EOL;
            $preAuthor = $author;


        }

        //
        $c = $this->getContent($dir . $href);
        $globleStr .= $c;


        // 最后一个
        if($element === end($eles)) {
            file_put_contents('/Users/mwh/sphinx/songci/source/' . $idx++ . '.rst', $globleStr);
        }
    }
}

private function getAuthor($file){
    $html = str_get_html(file_get_contents($file));
    $eles = $html->find("p");
    $authorList = [];
    foreach ($eles as $ele) {
        $authorList[] = $ele->plaintext;
    }
    return $authorList;
}

private function getContent($file){
    $html = str_get_html(file_get_contents($file));
    $title = $this->extractTitle($html);
    $main = $this->extractMain($html);
    $zhu = $this->extractZhu($html);
    return $title . $main . $zhu . PHP_EOL . PHP_EOL . PHP_EOL;
}

private function extractTitle($html){
    $str = PHP_EOL;
    $title = $html->find("p.title1", 0)->plaintext;
    $title = preg_replace("/[\x{2460}-\x{2471}]+/u"," [#]_ ", $title);
    $str .= str_replace('　', '' , $title) . PHP_EOL . '--------------------';
    return $str . PHP_EOL . PHP_EOL;
}

private function extractMain($html){
    $str = PHP_EOL;
    $text = $html->find("p.normaltext4", 0)->plaintext;
    $text = preg_replace("/[\x{2460}-\x{2471}]+/u"," [#]_ ",$text);
    $str .= str_replace('　　', PHP_EOL . PHP_EOL , $text);
    return $str . PHP_EOL . PHP_EOL;
}

private function extractZhu($html){
    $str = PHP_EOL;
    $ele = $html->find("p.normaltext8", 0);
    if($ele) {
        $str .= '.. rubric:: 注释' . PHP_EOL;
        $str .= preg_replace("/[\x{2460}-\x{2471}]+/u", PHP_EOL . ".. [#] ", $ele->plaintext);
        $str .= PHP_EOL . PHP_EOL;
    }
    return $str;
}

```


