webmagic-scripts
======

## goal:
It makes it possible to use a simple script to write the way reptiles, thereby providing a flow of scripts for some common scenarios.
If someone has already written a script, then you just use it!

## Examples:
For example: I need to catch github repository data, you can write a script (javascript):

```javascript
var name=xpath("//h1[@class='entry-title public']/strong/a/text()")
var readme=xpath("//div[@id='readme']/tidyText()")
var star=xpath("//ul[@class='pagehead-actions']/li[1]//a[@class='social-count js-social-count']/text()")
var fork=xpath("//ul[@class='pagehead-actions']/li[2]//a[@class='social-count']/text()")
var url=page.getUrl().toString()
if (name!=null){
    println(name)
    println(readme)
    println(star)
    println(url)
}

urls("(https://github\\.com/\\w+/\\w+)")
urls("(https://github\\.com/\\w+)")
```

Then use webmagic load and start it without downloading dependencies, write code, the process of implementation.
Already the console version, download the [http://code4craft.qiniudn.com/webmagic-console.tar.gz](http://code4craft.qiniudn.com/webmagic-console.tar.gz).

After decompression, use the following command:

```bash
	java -jar -Dfile.encoding='utf-8' webmagic-console.jar -f file-name-script [-l language, by default javascript] [-t theads] [-s interval, milliseconds] url1 url2 …
```

For example, for github this script, I can perform:

```bash
	java -jar -Dfile.encoding='utf-8' webmagic-console.jar -f github.js -t 2 -s 0 https://github.com/code4craft
```

This section is currently using Java ScriptEngine mechanism is completed.

## Language:

Because the broad selection of javascript user plane. Languages currently supported ruby, ruby chosen because DSL ruby syntax for writing more concise:

```ruby
name= xpath "//h1[@class='entry-title public']/strong/a/text()"
readme = xpath "//div[@id='readme']/tidyText()"
star = xpath "//ul[@class='pagehead-actions']/li[1]//a[@class='social-count js-social-count']/text()"
fork = xpath "//ul[@class='pagehead-actions']/li[2]//a[@class='social-count']/text()"
url=$page.getUrl().toString()

puts name,readme,star,fork,url unless name==nil

urls "(https://github\\.com/\\w+/\\w+)"
urls "(https://github\\.com/\\w+)"
```

Multilingual by parameter -l distinguish, for example, the implementation of the ruby script requires:

	java -jar -Dfile.encoding='utf-8' webmagic-console.jar -f github.rb -t2 -s0 -l ruby https://github.com/code4craft

This feature is still in testing stage. Welcome to participate and comment.