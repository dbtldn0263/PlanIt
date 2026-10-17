# PlanIt — 학습 계획 앱 (팀 프로젝트)

> 이 저장소는 팀 프로젝트 [Planit-TeamProject/Planit-InitAuthGeneratingAndMain-main-Delphox](https://github.com/Planit-TeamProject/Planit-InitAuthGeneratingAndMain-main-Delphox)의 포크입니다.
> 아래는 제가(유시우) 담당한 부분에 대한 설명입니다.

## 개요

책의 목차를 바탕으로 학습 계획(캘린더)을 자동으로 만들고, 매일의 진행 상황을 관리하는 웹 앱입니다.
AI 챗봇에게 질문하거나, 공부한 범위에서 나오는 퀴즈로 이해도를 확인할 수 있습니다.

- 기간: 2026년 8월 ~ 10월
- 인원: 5명
- 담당: 체크리스트, 챗봇, 퀴즈봇, 마이페이지

## 시스템 구성

```
React + Vite (프론트엔드)
 ├─ Spring Boot :8080  체크리스트 API   ← 담당
 ├─ FastAPI     :8000  챗봇             ← 담당
 └─ Spring Boot :8081  인증·퀴즈 API    ← 퀴즈 부분 담당
        │
 Cloud Firestore / Firebase Auth / Gemini API
```

팀 공통 규칙으로 `memberId = Firebase uid`, `studyPlanId = "plan-" + uid`로 정해, 서로 다른 API에서도 같은 학습 계획 데이터를 참조할 수 있게 했습니다.

## 담당 기능

### 1. 체크리스트 (Spring Boot)
`Planit-Web-Checklist-main/`
- 학습 항목의 진행률을 0·25·50·75·100% 5단계로 기록하는 REST API
- 본인 외에는 항목을 수정할 수 없도록 검사
- '오늘 학습 마무리'는 하루 1회만 (이미 기록이 있으면 새로 만들지 않음)
- MySQL/JPA에서 Cloud Firestore로 이전
- JUnit·Mockito로 서비스 계층 테스트 작성

### 2. 챗봇 (FastAPI + Gemini)
`chat_call.py`, `server.py`
- 그날의 학습 항목을 맥락으로 전달해, 학습 내용을 반영한 답변 생성
- 시스템 프롬프트로 답변 길이와 말투 제어
- 일시적 오류(5xx)는 대기 시간을 늘려 가며 최대 5회 재시도
- 하루 사용 횟수를 Firestore로 관리하고, Gemini 한도(429)와도 맞춤

### 3. 퀴즈봇 (Spring Boot + Gemini)
`Planit-Web-Auth-Plan-Quiz-master/.../quiz/`
- 진행률 75% 이상인 학습 항목에서 기초·응용 2문제를 자동 생성
- Gemini에 JSON 스키마를 지정해 출력 형식을 안정화
- 생성에 실패하면 재시도하고, 그래도 실패하면 대체 문제로 전환
- 정답은 제출 전까지 보여주지 않고, 중복 제출은 409로 거부

### 4. 마이페이지 (React + Firebase)
`frontend/src/screens/MyPageScreen.jsx`
- 이름 변경 (Firestore와 Firebase Auth 모두 갱신)
- 비밀번호 재설정 메일 발송
- 탈퇴 시 학습 데이터·인증 계정·로그인 상태를 순서대로 삭제
- 유료 플랜이 필요한 프로필 사진 기능은 비용을 고려해 제외

## 사용 기술 (담당 부분)

Java (Spring Boot, JPA, JUnit) / Python (FastAPI) / React / Cloud Firestore / Firebase Auth / Gemini API

## 실행 방법

실행하려면 Firebase 서비스 계정 키와 Gemini API 키가 필요합니다 (저장소에는 포함하지 않았습니다).

```
# 환경 변수
GEMINI_API_KEY=...
# 루트에 firebase-service-account.json 배치

# Windows
start.bat
```

## 작성자

유시우 (劉 始禹) — GitHub: [@dbtldn0263](https://github.com/dbtldn0263)

ーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーー
日本語
# PlanIt — 学習計画アプリ（チーム開発）

> このリポジトリは、チーム開発プロジェクト [Planit-TeamProject/Planit-InitAuthGeneratingAndMain-main-Delphox](https://github.com/Planit-TeamProject/Planit-InitAuthGeneratingAndMain-main-Delphox) のフォークです。
> 以下は、私（ユ シウ）が担当した部分の説明です。

## 概要

本の目次から学習計画（カレンダー）を自動で作り、日々の進捗を管理するWebアプリです。
AIチャットボットへの質問や、学習した範囲からのクイズで理解度を確認できます。

- 期間：2026年8月〜10月
- 人数：5名
- 担当：チェックリスト、チャットボット、クイズボット、マイページ

## システム構成

```
React + Vite（フロントエンド）
 ├─ Spring Boot :8080  チェックリストAPI   ← 担当
 ├─ FastAPI     :8000  チャットボット      ← 担当
 └─ Spring Boot :8081  認証・クイズAPI     ← クイズ部分を担当
        │
 Cloud Firestore / Firebase Auth / Gemini API
```

チーム共通のルールとして、`memberId = Firebase の uid`、`studyPlanId = "plan-" + uid` とし、別々のAPIでも同じ計画データを参照できるようにしました。

## 担当機能

### 1. チェックリスト（Spring Boot）
`Planit-Web-Checklist-main/`
- 学習項目の進捗を 0・25・50・75・100% の5段階で記録するREST API
- 本人以外の項目は更新できないようにチェック
- 「今日の学習を終える」は1日1回のみ（すでに記録があれば新しく作らない）
- MySQL/JPA から Cloud Firestore へ移行
- JUnit・Mockito によるサービス層のテスト

### 2. チャットボット（FastAPI + Gemini）
`chat_call.py`, `server.py`
- その日の学習項目を文脈として渡し、学習内容を踏まえて回答
- システムプロンプトで回答の長さや振る舞いを制御
- 一時的なエラー（5xx）は待ち時間を延ばしながら最大5回再試行
- 1日の利用回数を Firestore で管理し、Gemini の上限（429）とも同期

### 3. クイズボット（Spring Boot + Gemini）
`Planit-Web-Auth-Plan-Quiz-master/.../quiz/`
- 進捗75%以上の学習項目から、基礎・応用の2問を自動生成
- Gemini に JSON スキーマを指定して出力を安定化
- 生成に失敗した場合は再試行し、それでも失敗したら代替の問題に切り替え
- 正解は提出まで表示せず、二重提出は 409 で拒否

### 4. マイページ（React + Firebase）
`frontend/src/screens/MyPageScreen.jsx`
- 名前の変更（Firestore と Firebase Auth の両方を更新）
- パスワード再設定メールの送信
- 退会時に学習データ・認証アカウント・ログイン状態を順に削除
- 有料プランが必要なプロフィール写真機能は、費用面から外す判断をした

## 使用技術（担当部分）

Java（Spring Boot, JPA, JUnit）／ Python（FastAPI）／ React ／ Cloud Firestore ／ Firebase Auth ／ Gemini API

## 実行方法

実行には Firebase のサービスアカウントキーと Gemini の API キーが必要です（リポジトリには含めていません）。

```
# 環境変数
GEMINI_API_KEY=...
# ルートに firebase-service-account.json を配置

# Windows
start.bat
```

## 作成者

ユ シウ（劉 始禹）— GitHub: [@dbtldn0263](https://github.com/dbtldn0263)
