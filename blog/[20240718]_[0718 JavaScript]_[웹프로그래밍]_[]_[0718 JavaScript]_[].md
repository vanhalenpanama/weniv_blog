```python

# method와 function의 차이


let obj = {
  show1: function() {
    console.log('show1() 메서드 호출');
  }
}

//화살표 함수
let obj = {
  show1: () => {
    console.log('show1() 메서드 호출');
  }
};


function show2() {
  console.log('show2() 함수 호출');
}

obj.show1(); //메서드
show2(); // 함수





#자바스크립트에는 파이썬 sorted()와 같은 function이 없습니다.

자바스크립트 배열 복사

const scores = [90,3,2,4]

const scores2 = scores.slice();








#  forEach()


forEach() 메소드는 배열의 각 요소에 대해 주어진 함수를 실행합니다. 이 때, 함수는 인자로 배열 요소, 인덱스를 받습니다. forEach() 메소드는 배열의 요소를 순환하면서 해당 요소를 함수로 전달하고, 이 함수가 각 요소에 대해 실행됩니다.
const arr = ['참외', '키위', '감귤'];
arr.forEach(function(item, index) {
  console.log(item, index);
	arr[index] = index;
});

// 결과
// 참외 0
// 키위 1
// 감귤 2







#  map()을 이용하여 데이터 뽑기

const data = [
    {
        "_id": "642ba3980785cecff3f39a8d",
        "index": 0,
        "age": 28,
        "eyeColor": "green",
        "name": "Annette Middleton",
        "gender": "female",
        "company": "KINETICA"
    },
    {
        "_id": "642ba398d0fed6e17f2f50c9",
        "index": 1,
        "age": 37,
        "eyeColor": "green",
        "name": "Kidd Roman",
        "gender": "male",
        "company": "AUSTECH"
    },
    {
        "_id": "642ba39827d809511d00dd8d",
        "index": 2,
        "age": 39,
        "eyeColor": "brown",
        "name": "Best Ratliff",
        "gender": "male",
        "company": "PRISMATIC"
    }
];

const ages = data.map((item) => item.age);







# hasOwnProperty()


hasOwnProperty() 메소드는 객체가 특정 프로퍼티를 가지고 있는지를 나타내는 불리언 값을 반환합니다.
const aespa = {
  members: ['카리나', '윈터', '지젤', '닝닝'],
  from: '광야',
	sing: function(){
		return "적대적인 고난과 슬픔은 널 더 popping 진화시켜!"
	}
};

console.log(aespa.hasOwnProperty('itzy')); //itzy는 없어서 false
console.log(aespa.hasOwnProperty('from'));//true 반환








```