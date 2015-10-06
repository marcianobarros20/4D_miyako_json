# 4D_miyako_json
Demo of json functions in <a href="https://github.com/miyako/4d-plugin-oauth">OAuth plugin</a>

If you are using a version of 4D below v14 and want to work with JSONs using Miyako's plugin here are some examples to get you going. 

First, I like to use consistent names when working with this. The two most common I use are '$root' and '$node'. Using the same names makes it easier for me to keep things straight. Now let's look at what those two names mean. 

Let's start by creating a basic JSON: 
~~~
$root:=JSON New 
$node:=JSON Append text ($root;"keyValue";"This is some text for the text node")
$node:=JSON Append long ($root;"keyValue";1)
$json:=JSON Export to text ($root;JSON_WITH_WHITE_SPACE)
JSON CLOSE ($root)
~~~
1) ​This creates a JSON object in memory and the variable $root is like a pointer to it. You can't do anything directly with $root except use it for other commands. ​
2) creates a 'node' in the JSON. In this case an element with key = 'keyValue' and the text as its value. 
3) creates a node, sets the key to 'keyValue' and the value as 12345.
4) writes $root to a text variable. 
5) closes $root and clears it from memory (always be sure to close)

And here is what $json looks like: 
~~~
{
  "keyValue" : "This is some text for the text node",
  "keyValue" : 1
}
~~~
I intentionally used the same keyValue in this example to illustrate a point about JSON - json is like a piece of paper you write things on. There are very few restrictions regarding *content*. There's nothing inconsistent about having two elements with the same key and different values. It may or may not be useful but it's not inconsistent. 

So this example created the json, appended a couple of nodes and exported the result. Let's change it up a little bit like so: 
~~~
$root:=JSON New 
$node:=JSON Append text ($root;"keyValue";"This is some text for the text node")
$json:=JSON Export to text ($root;JSON_WITH_WHITE_SPACE)

JSON SET LONG ($node;1)
$json2:=JSON Export to text ($root;JSON_WITH_WHITE_SPACE)

JSON CLOSE ($root)
~~~

​This time I'm using JSON SET LONG on the node created in line 2. I also called export twice to show what the effect is: 
```
$json = {
	"keyValue" : "This is some text for the text node"
}

$json1 = {
	"keyValue" : 1
}
```
​This illustrates that I can manipulate a node directly once I know what it is. This includes changing its type. Remember - JSON let's you do that. 

​These were trivial examples and you don't need a plugin to make them. But let's say you've got a large chunk of text to include. The plug in deals with escaping characters that will cause problems like double quotes, commas and colons. 

Dates are handled as text by the way. So there's no 'JSON append date'. String your date and time values and use text. 

Another oddity is boolean. Booleans are passed as 0 or 1. Yes even though it says 'JSON append bool' you still have to pass 1 or 0. I use the Num(boolean) to manage this. 

Now let's talk about READING a json. Let's assume $json is a text var containing a JSON. Our first move is to parse it: 
```
$root:=JSON parse text($json)
```
​Usually I already know something (or everything) about the contents of a json. From the example above I could call:
```
$node:=JSON get child by name($root;"keyValue")
```
​Now I can do something with the node like retrieve the value:​
```
$text:=JSON get text($node)
```
​It's a two step process to get the value: find the node then get the node value. 

Let's say I call this now: 
```
$root:=JSON parse text($json1) // this is the json with a longint value
$node:=JSON get text("keyValue")
$text:=JSON get text($node)
$long:=JSON get long($node)
```
The var $text = "1" and $long = 1. This is in keeping with the way javascript works too. Pretty cool. What happens if there is no key with the name I pass? 
```
$node:=JSON get text("notHere")
$text:=JSON get text($node)
$long:=JSON get long($node)
$text = "", $long = 0 and no errors are generated. 
```
##Let's talk about arrays
Javascript arrays are much different than 4D arrays because they aren't required to be specifically typed. So it's perfectly legal to make a js array of: 
```
[1,2,3,"some text","11",{"key":"value"},"anotherArray":[6,7,8]]​
```
4D would have none of this mixing of types - numbers, text, other jsons and another array all as elements? No! 

As a 4D dev you have to keep that in mind. When putting an array into a json it's not a big deal but when you try to read an array from a json there's no guarantee you can just read the array as is. 

​I'm going to begin by writing 4D arrays to a json. You can copy and paste this:
```
ARRAY TEXT($aData;3)
$aData{0}:="x"  //this element is skipped
$aData{1}:="a"
$aData{2}:="b"
$aData{3}:="c"

ARRAY LONGINT($aLong;3)
$aLong{1}:=100
$aLong{2}:=101
$aLong{3}:=120

$root:=JSON New 
$node:=JSON Append text array ($root;"array";$aData)
$json:=JSON Export to text ($root;JSON_WITHOUT_WHITE_SPACE)
  //  $json = {"array":["a","b","c"]}
$node:=JSON Append long array ($root;"longArray";$aLong)
$json:=JSON Export to text ($root;JSON_WITHOUT_WHITE_SPACE)
  // $json = {"array":["a","b","c"],"longArray":[100,101,120]}
$json2:=JSON Export to text ($node;JSON_WITHOUT_WHITE_SPACE)  // what if I export the node?
// $json2 = [100,101,120]
JSON CLOSE ($root)​
```
​​Notice I can export $node just like I usually do with $root. The lesson here is $node is simply another JSON and can be treated as such. 

​This example gives us a json with 2 members both of which are arrays. In 4D land the arrays are different types in JS land they are just two arrays. 

##Now let's go about reading them
Let's say we acquired the json text into the $json variable. We need to parse it and see what's there.
```
ARRAY TEXT($aNames;0)
ARRAY TEXT($aNodes;0)
ARRAY LONGINT($aTypes;0)

$root:=JSON Parse text ($json)
JSON GET CHILD NODES ($root;$aNodes;$aTypes;$aNames)
JSON CLOSE ($root)
```
The arrays give us the key names, the node and the type. 
​$aNames{1}="array", $aNames{2}="longArray"​
​   Types:
   1 = json member - text
   2 = json member - long
   3 = json member - boolean
   4 = array of elements
   5 = json element of more elements

​So I've parsed the json and I want to work with 'longArray'. Here's how I get $node for it: 
```
$root:=JSON Parse text ($json)
JSON GET CHILD NODES ($root;$aNodes;$aTypes;$aNames)

$i:=Find in array($aNames;"longArray")
If ($i>0)
$node:=$aNodes{$i}
... do more stuff
end if​
```
##This is enough to get you going
I put together a demo database including the plugin, some of these examples and a couple of examples I found from Miyako in various places. I included a couple of utility methods I find really helpful:
```
​Json_extract_node(json text; key; pointer to var)
  // $1 is an unparsed json
  // $2 is the name (key) you want to extract
  // $3 is pointer to a variable of the type the data is
```
Useful for pulling out top level key/values. Coerces the value into whatever data type $3 is. 
```
JSON_parse_toFormObjects(json text)
```
Parses the json then steps through the top level members. If there is a form object with the same name it sets the form object to the key value. 

​These methods are really handy for quickly managing small, simple json. More complex and involved ones require more tinkering with. 
