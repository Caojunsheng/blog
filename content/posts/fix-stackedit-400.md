stackedit在弹出网页转到github授权界面的时候，在你点授权之前，F12打开console，输入下边的JS code：

```
window.XMLHttpRequest =  class MyXMLHttpRequest extends window.XMLHttpRequest {
open(...args){
if(args[1].startsWith("https://api.github.com/user?access_token=")) {
  // apply fix as described by github
  // https://developer.github.com/changes/2020-02-10-deprecating-auth-through-query-param/#changes-to-make
  
  const segments = args[1].split("?");
  args[1] = segments[0]; // remove query params from url
  const token = segments[1].split("=")[1]; // save the token
  
  const ret = super.open(...args);
  
  this.setRequestHeader("Authorization", `token ${token}`); // set required header
  
  return ret;
}
else {
  return super.open(...args);
}
}
}

```

这段代码会重写stackedit的API请求，可以成功解决400的问题，成功shouquan

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTUyNDEyNDM1N119
-->