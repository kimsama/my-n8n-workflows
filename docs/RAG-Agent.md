# n8n Workflow Documentation for RAG Agent

## Directory Listing Command
```bash
ls -p {{ $json.directory }} | grep -v / || true; \
echo "==="; \
ls -p {{ $json.directory }} | grep / || true;
```
해당 명령어는:
1. 파일만 나열 (`grep -v /`)
2. 구분선 출력 (`===`)
3. 디렉토리만 나열 (`grep /`)

### 재귀적 파일 검색
특정 확장자(.htm)를 가진 파일을 재귀적으로 검색하려면:
```bash
find {{ $json.directory }} -type f -name "*.htm" -print || true; \
echo "==="; \
find {{ $json.directory }} -type d -print || true
```

## 변수 참조
- `{{ $json.directory }}`: n8n의 워크플로우에서 이전 노드로부터 전달받은 데이터의 directory 속성을 참조
- 예: `http://blah.com/{{$node["Function-Transform"].json["id"]}}` - 특정 노드의 출력값 참조

## 파일 처리 워크플로우

### 1. Function 노드
파일 목록을 처리하는 기본 코드:
```javascript
return $json.files.map(file => {
    return {
        json: {
            filePath: file,
            file_id: file.split('/').pop().split('.')[0]
        },
        binary: {
            data: null  // Read Binary File 노드용 바이너리 데이터 필드
        }
    };
});
```

HTML 파일만 필터링하는 코드:
```javascript
return $json.files
    .filter(file => file.toLowerCase().endsWith('.html') || file.toLowerCase().endsWith('.htm'))
    .map(file => {
        return {
            json: {
                filePath: file,
                file_id: file.split('/').pop().split('.')[0]
            },
            binary: {
                data: null
            }
        };
    });
```

### 2. Read Binary File 노드
- File Path 설정: `{{ $json.filePath }}`

### 3. Extract from HTML 노드
- Format을 'Text'로 설정하여 일반 텍스트로 파일 내용 읽기

### 4. Default Data Loader 노드
Type of  Data:
- JSON 

Mode:
- Load Specific Data

Data:
- {{$json.data}}

Options 설정:
- Name: `file_id`
- Value: `{{ $json.file_id }}` 또는 다른 파일 식별자

## 주의사항
1. 바이너리 데이터 처리시 binary 필드 필요
2. 파일 경로와 접근 권한 확인
3. 적절한 파일 형식 지정 (HTML vs Text)