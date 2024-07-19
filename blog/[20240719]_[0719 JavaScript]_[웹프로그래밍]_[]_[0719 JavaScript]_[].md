```python

자바, 파이썬, 자바스크립트의 최상위 클래스는 object

클래스 선언



class Me2{
	constructor(name, address, phoneNum){
	this.name = name;
	this.address = address;
	this.phoneNum = phoneNum;
}
}

const Me2 = (name, address, phoneNum) => ({
  name,
  address,
  phoneNum,
});


### DOM 트리에 접근하기

document 객체를 통해 HTML 문서에 접근이 가능합니다. document는 브라우저가 불러온 웹페이지를 나타내며, DOM 트리의 진입점 역할을 수행합니다.

// 해당하는 Id를 가진 요소에 접근하기
document.getElementById();

// 해당하는 모든 요소에 접근하기
document.getElementsByTagName();

// 해당하는 클래스를 가진 모든 요소에 접근하기
document.getElementsByClassName();

// css 선택자로 단일 요소에 접근하기
document.querySelector("selector");

// css 선택자로 여러 요소에 접근하기
document.querySelectorAll("selector");



isNaN()은 판정이 너무 관대해서 쓸모가 없고 Number.isNaN()을 써야 좀 더 정확한 결과값을 받을 수 있다.


```