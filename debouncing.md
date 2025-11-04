//context 
/*const user = {
  name: "Ravi",
  greet: function() {
    console.log("Hello, I am", this.name);
  }
};

user.greet();  //"Hello, I am Ravi"

const debouncedGreet = debounce(user.greet, 500);
debouncedGreet(); //Hello, I am undefined

//so use  ---let context = this;

//arguments
function printAll() {
  console.log(arguments);
}

printAll(10, 20, 30); //[10, 20, 30]

*/


import React from "react";


const A = () => {
  
let counter = 0

const getData = () => {
  console.log('fetch Data', counter++)
}

const debounce = function (fn, d) {
  let timer

  return function () {
    let context = this,
    arg = arguments
    console.log("arg",arg)
    console.log('arguments',arguments)
    clearTimeout(timer)
    timer = setTimeout(() => {
      getData.apply(context, arguments)
    }, d)

  }
}

 

const click = debounce(getData, 300)
  return (
    <>
      <button onClick={click}> click</button>
    </>
  );
};

export default A;
