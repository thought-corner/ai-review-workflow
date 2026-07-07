# ai-review-workflow

개인용 Claude PR 리뷰 중앙 레포. 리뷰 로직/프롬프트를 여기 한 곳에서만 관리하고,
다른 레포들은 이 워크플로우를 호출만 한다. MAX 구독 한도로 동작(API 키 불필요).

## 구조
```
.github/workflows/claude-review.yml   # 재사용 워크플로우 (외부 레포가 호출하는 진입점)
prompts/review.md                     # 리뷰 프롬프트 (= 진짜 '코드', 여기만 고치면 전체 반영)
consumer-example/review.yml           # 소비 레포에 복붙할 호출 파일 샘플
```

## 최초 세팅 (한 번만)

### 1. MAX 구독으로 OAuth 토큰 발급 (로컬 PC)
```bash
npm install -g @anthropic-ai/claude-code
claude               # MAX 계정으로 로그인
claude setup-token   # 브라우저 OAuth → 1년짜리 토큰이 출력됨
```
출력된 토큰을 복사한다. (`/login` 세션 토큰은 몇 시간 만에 만료되므로 반드시 `setup-token` 사용)

### 2. 시크릿 등록
리뷰받을 **각 소비 레포**의 Settings → Secrets and variables → Actions → New repository secret
- Name: `CLAUDE_CODE_OAUTH_TOKEN`
- Value: 위에서 복사한 토큰

> 레포가 여러 개면 무료 GitHub Organization을 만들어 Organization Secret 으로 한 번만
> 등록하면 `secrets: inherit` 로 전 레포에서 공유된다.

### 3. 이 중앙 레포에 v1 태그 달기
```bash
git add . && git commit -m "init review workflow"
git push
git tag v1 && git push origin v1
```
로직을 바꿔 재배포/롤백할 땐 태그를 옮긴다:
```bash
git tag -f v1 && git push -f origin v1
```

### 4. public / private
public 이면 소비 레포에서 프롬프트 체크아웃 시 토큰이 필요 없다(권장).
private 로 두려면 `claude-review.yml` 의 `token:` 줄 주석을 풀고 PAT 시크릿을 추가한다.

## 소비 레포에서 호출
소비 레포에 `.github/workflows/review.yml` 을 추가한다 (consumer-example/review.yml 참고). 끝.
