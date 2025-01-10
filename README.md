# Document

## Schema 외근
```JSON
{
	email: "newzen@email.com", // 사용자 이메일
	group: "CompCd", // 사용자 회사 코드
	recordType: "outside", // 외근 
	
	startDate: 2022-01-01T00:00:00, // 외근 시작
	endDate?: 2022-01-01T04:00:00, // 외근 종료
	desc?: "외근 사유",
	location?: Location // 외근 장소 Optional
}
```

## Schema 출장
```JSON
{
	email: "newzen@email.com", // 사용자 이메일
	group: "CompCd", // 사용자 회사 코드
	recordType: "busstrip", // 출장 
	
	startDate: 2022-01-01T00:00:00, // 출장 시작
	endDate: 2022-01-01T04:00:00, // 출장 종료	
	desc?: "출장 사유",
}
```

## Schema 연차
```JSON
{
	email: "newzen@email.com", // 사용자 이메일
	group: "CompCd", // 사용자 회사 코드
	recordType: "annual", // 연차
	annualType: "string ex) 연차, 반차, 반반차, 공가",
	annualValue: "string number ex) '1', '20'", // 연차사용 일 수 
	desc?: "연차 사용 이유"
	startDate: 2022-01-01T00:00:00, // 연차 시작
	endDate: 2022-01-01T014:00:00, // 연차 종료
}
```

-----------------------------------------
## 등록
### POST /record/add

### Body

+ 외근 등록
```JSON
{
	type: "outside",
	data: {
		// 시간을 포함한 Date 형식의 문자열
		startDate: '2024-01-01 13:00',
		endDate:  '2024-01-01 15:00',
		desc: '외근 사유',
		
		location: { // 데이터 형식 체크하지 않습니다
		}
	}
}
```

+ 출장 등록
```JSON
{
	type: "busstrip",
	data: {
		// 시간을 포함한 Date 형식의 문자열
		startDate: '2024-01-01 13:00',
		endDate:  '2024-01-01 15:00',
		desc: '외근 사유',
	}
}
```

+ 연차 등록
```JSON
{
	type: "annual",
	data: {
		// 시간을 포함한 Date 형식의 문자열
		startDate: '2024-01-01 13:00',
		endDate:  '2024-01-01 15:00',
		annualType: '연차', // 연차이름
		annualValue: '1' 
	}
}
```

### Result
+ 성공
```JSON
{
	resultCode: 'ok',
	data: ''
}
```
+ 실패
```JSON

{
	// DB에 저장된 데이터 중 startDate <= newData.startDate 이 면서
	// endDate 필드가 없는 데이터가 존재할 경우	
	resultCode: 'OutsideNotCompleted',
	comment: '퇴근처리 되지 않은 외근이 있습니다'
}

{
	// DB에 저장된 데이터 중 startDate < newData.startDate && newData.endDate <= endDate 존재할 경우
	resultCode: 'BusstripDateOverlap',
	comment: '기간이 겹치는 출장 데이터가 존재합니다'
}

{
	// DB에 저장된 데이터 중 startDate < newData.startDate && newData.endDate <= endDate 존재할 경우
	resultCode: 'AnnualDateOverlap',
	comment: '기간이 겹치는 연차 데이터가 존재합니다'
}

```

-------------------------------

## 조회

### GET /record/list

### Query
```JSON
?type=outside,annual,busstrip&from=2024-01-01&to=2024-01-01

from, to 조건
from,to에 만족하는 startDate 필드를 가진 record 찾습니다

데이터 형식은 `YYYY-MM-DD`, `YYYYMMDD`, `YYYY-MM-DD HH:MM`, `YYYY-MM-DDTHH:HH`
등의 `년월일시분` 구조입니다

ex) 
1. from=2024-01-01 13:00&to=2024-01-01
	1. 2024-01-01 13:00 <= startDate && startDate < 2024-01-02
2. from=2024-01-01&to=2024-01-01T18:00
	1. 2024-01-01 00:00 <= startDate && startDate < 2024-01-01 19:00

to 일자에 시간 조건이 포함되어 있을 경우 1시간을 더하고, 시간조건이 없을 경우 1일을 더하여 less then 비교를 진행 합니다
```

### Result
```JSON
{
	resultCode: 'ok',
	data: [
		{
			"_id": "66f3cd8ffde2e167ffeaaddd", // document key
			"email": "newzen00001@newzen.com", 
			"group": "8jeK1txj",
			"recordType": "outside",
			"startDate": "2024-01-03T06:00:00.000Z",
			// endDate 없을 경우 퇴근처리 되지 않은 데이터
			"createdAt": "2024-09-25T08:45:03.883Z",
			"updatedAt": "2024-09-25T08:45:03.883Z",
		},
		{
			"_id": "66f3a1c961d3bf440aeb2fcd", // document key
			"email": "newzen00001@newzen.com", 
			"group": "8jeK1txj",
			"recordType": "outside",
			"startDate": "2024-01-01T06:00:00.000Z",
			"endDate": "2024-01-01T08:00:00.000Z",
			
			"createdAt": "2024-09-25T08:45:03.883Z",
			"updatedAt": "2024-09-25T08:45:03.883Z",
		},
	]
}
```

------------------------

## 편집

### POST /record/edit

### Body

update 필드의 형식을 검증하지 않습니다. 몽고DB 내장 형식(Date 등) 을 사용하는 필드를 문자열로 변경 시 Date 관련 Query 실행이 안됩니다.(startDate, endDate 제외)
```JSON
{
	key: "66f3a1c961d3bf440aeb2fcd",
	update: {
		// 편집하거나 추가할 필드
		desc: "개인사정으로 이한 연차 사용",
		annualType: "0.5",		
		endDate: "2024-01-01 10:00:00"
	}
}
```

### Result
```JSON
{
	resultCode: 'ok',
	data: ''
}

{
	// update.startDate, update.endDate 필드가 존재할 시
	// 해당 문자열을 Date 타입으로 형변환 후 유효일자 여부 체크
	resultCode: 'InvalidDate',
	comment: '유효하지 않은 일자입니다'
}
```

-----------------------------------------------
## 삭제

### POST /record/remove

### Body
```JSON
{
	key: [ "66f3a1c961d3bf440aeb2fcd", "" ]
}
```

### Result
```JSON
{
	resultCode: 'ok',
	data: ''
}
```

--------------------------

# WORK365 API

조회는 근태 데이터 중 회사코드가 일치하는 데이터 조회
편집/삭제는 회사코드 일치 및 현재 사원 목록과 일치할 경우 실행

## 조회
### GET /attendance/record-all
	 관리자 권한 필요

### Query
```JSON
type: 'annual' or 'outside'
from: YYYY-MM-DD
to: YYYY-MM-DD
email: email1@newzen.com

ex)
?from=2024-01-01&2024-01-01&type=outside&email=newzen1@newzen.com
```

### Result
```JSON
{
	"resultCode": "ok",
    "comment": "성공",
    "data": [
        {
            "_id": "66f4ea03af1c49e0fcc02683",
            "email": "picnicqsenn@gmail.com", // 이메일
            "group": "BOhrQOfp", // 회사 코드
            "recordType": "outside", // 외근
            "startDate": "2024-01-01T06:00:00.000Z",
            "createdAt": "2024-09-26T04:58:43.224Z",
            "updatedAt": "2024-09-26T04:58:43.224Z",
            "__v": 0,
            "name": "TEST_NAME" // 이름
        }
    ]	
}
```


------------------------

## 편집
	관리자 권한 필요
	관리자가 일반 사용자 근태 편집 시 사용
	
### POST /attendance/record-edit

### Body
```JSON
{
	key: "66f3a1c961d3bf440aeb2fcd",
	update: {
		// 편집하거나 추가할 필드
	}
}
```

### Result
```JSON
// 성공
{
	resultCode: "ok",
	data: {
		modifiedCount: 1,
		matchedCount: 1
	}
}

{
	resultCode: "NotFoundEmp",
	comment: "사원정보를 찾을 수 없습니다"
}
//
```

--------------

## 삭제
	 관리자 권한 필요

### POST /attendance/record-remove

### Body
```JSON
{
	key: ["", ""]
}
```

### Result
``` JSON
{
	resultCode: "ok",
	data: {
		deletedCount: 1,
		matchedCount: 1,		
	}
}

```
