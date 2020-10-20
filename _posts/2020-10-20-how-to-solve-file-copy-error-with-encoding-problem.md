---
title:  "DB 인코딩 문제로 인한 파일복사 실패 시"
excerpt: "똑같은 희생자가 나오지 않기를 바라며"

categories:
  - DB
tags:
  - DB
---
입사 후 재인코딩 솔루션을 분석, 설계, 구현까지 3주 만에 한 뒤, 인코딩 문제로 한글 파일 복사에 실패하여 3주 간 삽질을 한 경험이 있습니다. 저뿐만이 아닌 주기적으로 겪는 연례행사라고 하는데, 이번 시간에는 해당 이슈의 문제점과 해결법을 문서화하여 똑같은 시행착오가 반복되지 않길 바라는 마음에 본 포스팅을 작성하게 되었습니다.

### 이슈
재인코딩 솔루션(reencd) 실행 과정 중 파일복사에서 한글파일명 영상 / 한글파일명 자막의 복사에 실패하며 V8 에러를 throw.

### 원인
전반적인 시스템 인코딩 환경은 UTF-8로 세팅이 되어 있으나 DB는 euc-kr을 사용하므로 한글이 테이블에 저장될 때 깨져서 들어감.

### 해결책
테이블에서 한글 파일명을 가져올 때 일부러 다른 인코딩을 선택하여 깨진 상태로 가져오면, 놀랍게도 에러가 나지 않는다.

### 문제가 된 소스
SELECT schedule_no, osp_id, cont_id, seq_no, encd_resolution, cont_server_id, cont_server_ip, cont_full_path, cont_file_size, cont_smi_server_id, cont_smi_server_ip, cont_smi_full_path, default_hash, mgr_cd, `cont_file_name, cont_smi_file_name` FROM T_STRM_SCHEDULE_FREE WHERE osp_id = '" + osp_id + "' AND (value1 = 'REN' and status_cd = 'BB') ORDER BY schedule_no LIMIT 1;

### 수정된 소스
SELECT schedule_no, osp_id, cont_id, seq_no, encd_resolution, cont_server_id, cont_server_ip, cont_full_path, cont_file_size, cont_smi_server_id, cont_smi_server_ip, cont_smi_full_path, default_hash, mgr_cd, `CONVERT(cont_file_name USING koi8r) as cont_file_name, CONVERT(cont_smi_file_name USING koi8r) as cont_smi_file_name` FROM T_STRM_SCHEDULE_FREE WHERE osp_id = '" + osp_id + "' AND (value1 = 'REN' and status_cd = 'BB') ORDER BY schedule_no LIMIT 1;