---
title: Data Table을 사용하여 그리드 표현하기(서버 사이드)
date: 2022-11-03 14:54:00
categories: etc  
published: true 
tags:
- etc 
- DataTable  
- Library  
---


기존 사용중이던 그리드를 데이터테이블로 변경 해야 할 일이 있었다. 무료로 제공되는 기능들이 다양하고 사용이 쉬워 좋았다. 그리고 기존 사용중이던 그리드에 비해 코드만 보고도 어떤 기능을 어떻게 고치면 될 지 상대적으로 알아보기 쉬워서 자주 사용하게 될 것 같다. 
데이터가 많아지면 기본 페이징 방식으로는 속도가 저하되므로(그리드를 변경한 이유도 이 부분 떄문이다) 서버 사이드 페이징 방식으로 구현하였다. 

### 뷰 설정

```html
<table id="visitRequestDataTable">
	  <thead>
	      <tr>
	          <th>신청번호</th>
	          <th>사업소</th>
	          <th>방문자</th>
	          <th>소속기관</th>
	          <th>연락처</th>
	          <th>차량번호</th>
	          <th>결재상태</th>
	          <th>담당자</th>
	          <th>방문일</th>					            					            
	      </tr>
	  </thead>					   
</table>
```

위와 같이 헤더 부분만 id를 지정해서 만들어 주면 html 영역에서 할 일은 끝이다. 이후 스크립트 부분을 이렇게 추가해주면 된다.

```jsx
	var columns = ["request_id","workplace","visitor_name","department_name","cell_phone_no","car_num","is_approval","user_name","start_visit"];
  	var lang_kor = {
  			"zeroRecords" : "검색된 데이터가 없습니다.",
  			"emptyTable" : "데이터가 없습니다.",
  			"info" : "_START_ - _END_ (총 _TOTAL_ 명)",        
  			"infoEmpty" : "0명",
  			 "lengthMenu": "페이지당 줄수 _MENU_"
  	};
            $('#visitRequestDataTable').DataTable({
            	stateSave: true, //현재 상태(페이지 수, 표시 건수, 정렬상태) 보존 
              displayLength: 20,
              pagingType : "full_numbers",
              paging: true,
              lengthChange: true,                
              lengthMenu : [10,20,25,50,100],
	           	//dom : 'Bfrtip',        // 엑셀 다운로드 기능 추가
	           	//buttons: ["excel"],
              responsive: true,
              processing: true, 
              serverSide: true,
              bAutoWidth: false,
              language : lang_kor,
              order: [[0,"desc"], [8,"desc"]],
              ordering: true,
              searching: false,
              destroy: true,
              ajax : {
                    url:"/visit/selectNewVisitRequestList.do?"+params+"&columns="+columns,
                    type:"POST",
                },     
              columns : [
                    {"data": "request_id"},
                    {"data": "daedukName",orderable:false},
                    {"data": "visitor_name",orderable:false},                    
                    {"data": "department_name"},
                    {"data": "cell_phone_no",orderable:false},
                    {"data": "car_num"},
                    {"data": "is_approval",orderable:false},
                    {"data": "user_name",orderable:false},
                    {"data": "start_visit"},                    
                ], 
            	columnDefs: [             		
            		{ targets: '_all', className: 'visit_more_tac' }, 
            		{ targets: [2,5,7], width:90},
            		{ targets: [1], width:75}
            	]
            });
```

- `stateSave :` true 
다른 페이지로 이동했다 돌아와도 정렬이나 페이지 수 등의 상태를 유지시켜주는 기능이다.
- `pagingType :` 
[simple, simple_numbers, full, full_numbers] 네 종류가 있다. PREV/NEXT 만 표시할 지, 페이지 숫자도 함께 표시할 지 등을 정하는 옵션이다.
- `paging :` true 
페이징 옵션을 설정한다. false일 때, scroll Y 옵션을 같이 사용해주지 않으면 한 화면에 모든 row가 다 나오니 주의해야 한다.
- `displayLength :` 20 
처음 페이지에 접근했을 때 한 페이지에 보여줄 게시물의 갯수. lengthChange로 사용자가 변경할 수 있다.
- `lengthChange :` true
- `lengthMenu :` [10,20,25,50,100] 
리스트를 한 페이지에 몇 row씩 보여줄 지 선택 가능하도록 옵션을 제공한다.
- `dom :` 'Bfrtip',      
`buttons :` ["excel"]
엑셀, CSV, PDF 등 다운로드 기능을 제공한다. 전체 페이지 뿐만 아니라 검색 조건에 따른 다운로드도 가능함.
- `processing :` true, serverSide: true 
서버 사이드로 변경하기 위해 필요한 설정들
- `responsive :` true
반응형 지원 여부
- `bAutoWidth :` false,
- l`anguage :` lang_kor 
영어로 된 문구들을 원하는 내용으로 대체 가능하다.
- `order :`  [[0,"desc"], [8,"desc"]], ordering: true 
정렬 기능. 원하는 필드의 오름차순/내림차순 설정이 가능하고 특정 항목은 정렬이 안 되도록 제외도 가능하다.
- `searching :` false 
데이터테이블의 검색기능은 입력한 내용을 즉시 검색하여 반영하기 때문에 현재 프로젝트와 맞지 않아 사용하지 않고, parameter를 넘겨서 검색하는 방식으로 따로 구현했다.
- `destroy :` true 
버튼 클릭(검색이나 새로고침 등) 등의 이벤트가 발생했을 때 새로 데이터를 로드 해준다.
- `columns :` [ 
                    {"data": "request_id", orderable:false},
                ]
각 head 아래에 들어갈 데이터들의 이름을 맞춰준다. 정렬 여부도 여기에서 설정이 가능하다.
- `columnDefs:` [             		
            		{ targets: '_all', className: 'visit_more_tac' }, 
            		{ targets: [2,5,7], width:90},
            		{ targets: [1], width:75}
            	]
            });
컬럼 별 css설정이나 class 부여 등이 필요할 때 여기에서 지정하면 된다. render 옵션도 사용 가능.
  

<aside>  
📌 **var columns**를 만들어서 컬럼명을 한 번 더 명시한 이유는 정렬과 페이징을 서버에서 처리할 예정이기 때문이다. 
**params**는 검색어를 따로 받아서 URL과 함께 보낼 거라 추가해 둔 것.
</aside>

---

### VO

```java
public class WrapperVO {
	private int iTotalRecords;
	private int iTotalDisplayRecords;
	private List<?> data;
		
	
	public List<?> getData() {
		return data;
	}
	public void setData(List<?> data) {
		this.data = data;
	}
	public int getiTotalRecords() {
		return iTotalRecords;
	}
	public void setiTotalRecords(int iTotalRecords) {
		this.iTotalRecords = iTotalRecords;
	}
	public int getiTotalDisplayRecords() {
		return iTotalDisplayRecords;
	}
	public void setiTotalDisplayRecords(int iTotalDisplayRecords) {
		this.iTotalDisplayRecords = iTotalDisplayRecords;
	}
```

클라이언트로 값을 보낼 때 필요한 VO다. 데이터테이블은 data라는 리스트 안에 키값이 들어있어야 컬럼을 인식해서 매칭할 수 있다. 

```java
public class VisitVO {
	private String request_id; 
	... //기타등등 자기에게 필요한 도메인들 
	private String length;
	private String start;
	private String sortDir;
	private String sortColumn;
	
	public String getSortDir() {
			return sortDir;
		}
	
		public void setSortDir(String sortDir) {
			this.sortDir = sortDir;
		}
	
		public String getSortColumn() {
			return sortColumn;
		}
	
		public void setSortColumn(String sortColumn) {
			this.sortColumn = sortColumn;
		}
	
		public String getLength() {
			return length;
		}
	
		public void setLength(String length) {
			this.length = length;
		}
	
		public String getStart() {
			return start;
		}
	
		public void setStart(String start) {
			this.start = start;
		}

```

위 항목들은 서버사이드 페이징과 정렬에 필요하다. start는 시작 번호, length는 시작지점에서부터 몇개씩 끊어서 가져 올 지(limit), sortDir은 오름차순/내림차순 설정, sortColumn은 컬럼명 선택에 필요하다. 

---

### Controller

```java
@RequestMapping(value = "/visit/selectNewVisitRequestList.do")
    public String selectNewVisitRequestList(Model model, HttpServletRequest request, 
HttpServletResponse response, VisitVO vo) throws Exception {
   	
		String sortColumn = request.getParameter("order[0][column]"); //컬럼 위치
		int sortColumnNum = Integer.parseInt(sortColumn); //받아온 컬럼 위치를 숫자로 변경 
		String[] columnName = request.getParameter("columns").split(","); //받아온 컬럼명을 배열에 저장
		vo.setSortDir(request.getParameter("order[0][dir]"));  //desc, asc 
		vo.setSortColumn(columnName[sortColumnNum]);     	

	  //그리드 service 호출
	    List<VisitVO> list = visitService.selectNewVisitRequestGridList(vo);
	    
	  //페이징 고려하지 않은 전체 카운트 수 호출 
	  int totalCount = visitService.selectNewVisitRequestListCount(vo);

	  //클라이언트로 값 전송을 위해 VO로 감싸기
		WrapperVO rtnVO = new WrapperVO();
		rtnVO.setData(list);
		rtnVO.setiTotalRecords(totalCount); //실제 전체 데이터
		rtnVO.setiTotalDisplayRecords(totalCount); //필터링 된 전체 데이터 
		
		//json으로 parsing 
		ObjectMapper objectMapper = new ObjectMapper();
		String jsonString = objectMapper.writeValueAsString(rtnVO);
		
		//이건 json을 ajax로 넘기도록 따로 만든 util이라 참고하지 않아도 됨. 
		// 알아서 ajax로 데이터를 넘길 것 
		ResponseHelper.makeAjaxMessage(response, jsonString);
	  return null;
}
```

---

### Service

```java
public List<VisitVO> selectNewVisitRequestGridList(VisitVO vo) throws SQLException {

      List<VisitVO> visitList = visitDAO.selectNewVisitRequestList(vo);
      for (VisitVO o : visitList) {
        	o.setVisitor_name(AriaUtil.decrypt(o.getVisitor_name()));
        	o.setUser_name(AriaUtil.decrypt(o.getUser_name()));
        	if("".equals(o.getCar_num()) || o.getCar_num()==null){
        		o.setCar_num("-");
        	}
        	o.setCell_phone_no(AriaUtil.decrypt(o.getCell_phone_no())); 
					//데이터 암호화 등의 처리가 필요한 동작 수행 
        }

        return visitList;
    }
```

---

### DAO

```java
public List<VisitVO> selectNewVisitRequestList(VisitVO vo) {   	
    	List<VisitVO> list =  sqlMapClientTemplate.selectList("visit.visit.selectNewVisitRequestList", vo);
        return list;
    }
```

---

### XML  
```sql  
<select id="selectNewVisitRequestList" parameterType="com.visit.vo.VisitVO" resultType="com.visit.vo.VisitVO">
		SELECT 필요한 필드 FROM 조회할 테이블 WHERE 조건 
     	ORDER BY ${sortColumn} ${sortDir}
     <if test="start != null and length != null">
         LIMIT ${start} , ${length}
     </if>
</select>
```  

---

### 결과화면  

![이미지](https://i.imgur.com/4Sm8taw.png)

