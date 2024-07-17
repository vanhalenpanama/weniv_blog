```python

자바스크립트에서 0 은 false 10 은 true


크롬 개발자도구에서 우클릭 붙여넣기하려면 
allow pasting을 먼저 입력한다






-화살표 함수 선언........................

// 매개변수 지정 방법
    () => { ... } // 매개변수가 없을 경우
     x => { ... } // 매개변수가 한 개인 경우, 소괄호를 생략할 수 있다.
(x, y) => { ... } // 매개변수가 여러 개인 경우, 소괄호를 생략할 수 없다.


......................................
if-else 문

let x = 3;
let y = 7;

if(x == y){
  console.log('if문으로 실행되었습니다.');
} else{
  console.log('else문으로 실행되었습니다.');
}



...................................
switch문


let price = 0;
let menu = '카페라떼'; // 메뉴를 바꿔보세요!

switch (menu) {
		case '아메리카노':
				price = 4000;
	  case '카페라떼':
				price = 5000;
}


....................

while 문


while (조건식) {
  // 조건식이 참일 때 실행될 코드
}


let num = 0;

while (num < 11) {
  document.write(num, '<br>');
  num += 1;
}




...........................


배열 (Array)




push()와 pop()


push() 메소드는 배열의 끝에 요소를 추가하고 길이를 반환합니다, pop() 메소드는 배열의 마지막 요소를 꺼내어 반환합니다. 꺼낸 요소는 배열에서 제외됩니다.

const arr = [1, 2, 3];
arr.push(4);
console.log(arr); // [1, 2, 3, 4]
arr.pop();
console.log(arr); // [1, 2, 3]




shift()와 unshift()


shift() 메소드는 배열에서 첫 번째 요소를 꺼내고 반환합니다, unshift() 메소드는 배열의 첫 번째 요소로 새로운 요소를 추가합니다

const myArray = ["사과", "바나나", "수박"];
myArray.shift();
console.log(myArray); 
myArray.unshift("오이", "배");
console.log(myArray);


splice()

splice() 메소드는 배열의 요소를 추가, 제거 또는 교체합니다.

메소드는 3개의 전달인자를 받습니다. 첫 번째  인자는 삭제나 추가를 시작할 인덱스입니다. 두 번째 인자는 삭제할 요소의 개수입니다. 세 번째 인자 부터는 추가할 요소들입니다. 추가할 요소가 없다면 생략할 수 있으며 이때는 요소를 삭제만 하게됩니다.


const arr = [1, 2, 3];
arr.splice(1, 0, 4);
console.log(arr); // [1, 4, 2, 3]
arr.splice(2, 1, 5);
console.log(arr); // [1, 4, 5, 3]



slice()

slice() 메소드는 배열에서 요소들을 추출하여 새로운 배열로 반환하는 메서드입니다.

메소드는 2개의 전달인자를 받습니다. 첫 번째 인자는 추출을 시작할 인덱스입니다. 두 번째 인자는 추출을 끝낼 인덱스입니다. 추출할 요소는 첫 번째 인자에서 시작하여, 두 번째 인자에서 바로 이전 요소까지입니다. 두 번째 인자는 생략 가능하며, 생략하거나 배열의 길이보다 큰 값을 전달하면 배열의 끝까지 추출합니다.


const myArray = ["apple", "banana", "cherry", "durian", "elderberry"];
console.log(myArray.slice(1, 4)); 
console.log(myArray.slice()); 
console.log(myArray.slice(0, 10));



```