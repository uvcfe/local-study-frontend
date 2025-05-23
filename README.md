# local-study-frontend
지역 스터디 매칭 웹앱 프론트엔드



```
- mock 데이터 json 으로 분리

- Sidebar 컴포넌트 생성
- Sidebar 컴포넌트 App.vue 에 전역으로 적용
- Sidebar 에서 카테고리 선택 시 active 활성화
- Sidebar 에서 카테고리 선택 후 카테고리에 해당하는 데이터 목록에 출력
- Sidebar 에서 카테고리 선택 후 목록에서 항목 선택 시 해당하는 데이터 StudyDetail로 전달
- Sidebar 에서 카테고리 선택 후 목록에서 항목 선택 시 active 활성화 유지
- Sidebar 에서 신청스터디,운영스터디 목록에서 항목 선택 시 active 활성화 유지

- 마이페이지 클릭 시 마이페이지 해당 메뉴 제거
- 마이페이지 클릭 시 뒤로가기 버튼 제거 후 내정보 수정으로 이동

- 스터디 목록에서 카드 갯수를 가로로 기본 4개 적용 후 브라우저 사이즈에 따라 반응형 대응으로 변경

- 메인 목록 드롭다운과 신청운영스터디 목록 버튼 사이즈 동일하게 적용
- 마이페이지에 헤더 컴포넌트 삭제 (전역 해더와 중복)

- 전체 목록에서 카드 이미지 썸네일 로드 전에 스켈레톤 로딩 이미지 추가
- 전체 목록에서 카드 클릭 후 상세 페이지에서 이미지 썸네일 로드 전에 스켈레톤 로딩 이미지 추가
```

***


- 포크&원본 리포 코드 가져오기  
  ```
  [1] 포크한 리포에서 작업 중이고, 원본 저장소(hsdevB)의 최신 코드를 가져오고 싶을 경우

  1. 원본 저장소를 upstream으로 등록 (한 번만)
  git remote add upstream https://github.com/hsdevB/local-study-frontend.git

  2. 원본 저장소의 최신 코드 가져오기
  git fetch upstream

  3. 내 브랜치(fe)에 원본의 main을 병합
  git checkout fe
  git merge upstream/main

  4. 변경사항 푸시 (필요 시)
  git push origin fe



  [2] 원본 저장소에서 작업 중이고, 다른 포크(uvcfe)의 fe 브랜치 코드를 가져오고 싶을 경우

  1. 포크된 저장소를 remote로 등록
  git remote add uvcfe https://github.com/uvcfe/local-study-frontend.git

  2. 포크된 저장소의 브랜치 가져오기
  git fetch uvcfe

  3. 내 브랜치(main 등)에 포크의 fe 브랜치 병합
  git checkout main
  git merge uvcfe/fe
  ```