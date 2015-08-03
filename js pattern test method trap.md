<script >
if (!String.prototype.trim) {
  String.prototype.trim = function() {
    return this.replace(/^\s+|\s+$/g, '');
  };
}

function formatDate(inputDate, isFull, separator){
  if (!inputDate) {
    return '';
  }
  
  var output = [];
  
  if (isFull) {
    output.push(inputDate.getFullYear());
  }
  
  var month = (inputDate.getMonth() + 1);
  output.push((month < 10) ? '0' + month : month);
  
  var date = inputDate.getDate();
  output.push((date < 10) ? '0' + date : date);
  
  return output.join(separator || '-');
}

function formatTime(inputDate, addSeconds){
  if (typeof addSeconds === 'undefined') {
    addSeconds = false;
  }
  
  var output = [];
  
  var hours = inputDate.getHours();
  output.push((hours < 10) ? '0' + hours : hours);
  
  var minutes = inputDate.getMinutes();
  output.push((minutes < 10) ? '0' + minutes : minutes);
  
  if (addSeconds) {
    var seconds = inputDate.getSeconds();
    output.push((seconds < 10) ? '0' + seconds : seconds);
  }
  
  return output.join(':');
}

function setDateTime(method, data, isSetDate){
  var datePattern = /\d{4}-\d{2}-\d{2}/ig, timePattern = /\d{2}:\d{2}/ig;
  var oldDateTime = method().trim(), now = new Date();
  if (!oldDateTime) {
    oldDateTime = formatDate(now, true) + ' ' + formatTime(now, false);
  }
  
  if (isSetDate) {
    if (datePattern.test(data)) {
      **datePattern.lastIndex = 0;**
      if (datePattern.test(oldDateTime)) {
        oldDateTime = oldDateTime.replace(datePattern, data);
      } else {
        oldDateTime = data + ' ' + oldDateTime;
      }
    } else {
      if (data) {
        oldDateTime = data + ' ' + oldDateTime;
      } else {
        oldDateTime = oldDateTime.replace(datePattern, '');
      }
    }
  } else {
    if (timePattern.test(data)) {
      **timePattern.lastIndex = 0;**
      if (timePattern.test(oldDateTime)) {
        oldDateTime = oldDateTime.replace(timePattern, data);
      } else {
        oldDateTime += (' ' + data);
      }
    } else {
      if (data) {
        oldDateTime += (' ' + data);
      } else {
        oldDateTime = oldDateTime.replace(timePattern, '');
      }
    }
  }
  
  method(oldDateTime.trim());
}

var time = '';
setDateTime(function (data) {
  if (!data) {
    return '';
  }
  time = data;
}, '2015-01-23', true);

setDateTime(function (data) {
  if (!data) {
    return '';
  }
  time = data;
}, '18:44', false);

console.log(time); //2015-07-31 18:44
</script>

今天在写如上代码的时候，提炼出来的一段代码。

问题描述：
当用同一个pattern解析两个不同的变量，但都是正确日期格式的时候，第二个却始终返回false。最后调试了半天，才发现是下面这个原因造成的。

原因如下：test()方法的返回值为Boolean类型，如果字符串中存在该正则表达式模式的匹配，就返回true，否则false。
值得注意的是，每次执行test()函数都只查找最多一个匹配，如果找到就立即返回true，否则返回false。
如果为正则表达式设置了全局标志(g)，test()函数仍然只查找最多一个匹配，不过我们再次调用该对象的test()函数就可以查找下一个匹配。

其原因是：如果regExpObject带有全局标志g，test()函数不是从字符串的开头开始查找，而是从属性regExpObject.lastIndex所指定的索引处开始查找。该属性值默认为0，所以第一次仍然是从字符串的开头查找。当找到一个匹配时，test()函数会将regExpObject.lastIndex的值改为字符串中本次匹配内容的最后一个字符的下一个索引位置。当再次执行test()函数时，将会从该索引位置处开始查找，从而找到下一个匹配。

因此，当我们使用test()函数执行了一次匹配之后，如果想要重新使用test()函数从头开始查找，则需要手动将regExpObject.lastIndex的值重置为 0。如果test()函数再也找不到可以匹配的文本时，该函数会自动把regExpObject.lastIndex属性重置为 0。
