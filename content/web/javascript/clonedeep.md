---
title: 深克隆 
date: "2022-05-01T22:12:03.284Z"
description: "深克隆"
---

# cloneDeep

```js
function getType(data) {
  return Object.prototype.toString.call(data).slice(8, -1);
}

function cloneDeep(data) {
  let basicData = ["number", "string", "boolean"];
  let ifBasic = basicData.some((ele) => ele === typeof data);
  if (ifBasic) {
    return data;
  }

  if (getType(data) === "Object") {
    let temp = {};
    for (let key in data) {
      temp[key] = cloneDeep(data[key]);
    }
    return temp;
  }

  if (getType(data) === "Array") {
    return data.map((ele) => cloneDeep(ele));
  }
}

let a = {
  a: 2,
  b: {
    c: 3,
  },
  d: [1, 2, 3],
};
let b = cloneDeep(a);
console.log(a, b);
a.b.c = 4;
b.d.push(4);
console.log(a, b);
```
