# 5.3 람다 함수 스케줄링

> _EventBridge 규칙을 설정하여 람다 함수를 매분마다 실행한다_

① 일정 표현식으로 이벤트 규칙 생성

```bash
# 일정 표현식을 rate 형식으로 구현할 경우
RULE_ARN=$(aws events put-rule --name "EveryMinuteEvent" \
	--schedule-expression "rate(1 minute)")

# 일정 표현식을 cron 형식으로 구현할 경우
RULE_ARN=$(aws events put-rule --name "EveryMinuteEvent" \
	--schedule-expression "cron(* * * * ? *)")
```

② EventBridge 서비스가 호출할 수 있도록 람다 함수에 권한 추가

```bash
aws lambda add-permission --function-name $LAMBDA_ARN \
	--action lambda:InvokeFunction --statement-id events \
	--principal events.amazonaws.com
```

③ 생성한 규칙의 대상으로 람다 함수 추가

```bash
aws events put-targets --rule EveryMinuteEvent \
	--targets "Id"="1","Arn"="$LAMBDA_ARN"
```

---

**🥕 유효성 검사** : CloudWatch Logs 로그 그룹을 통해 함수가 60초마다 호출되는지 확인

```bash
aws logs tail "/aws/lambda/AWSCookbook503Lambda" --follow --since 10s
```

**🥕 참고**

⍢ 리소스 프로비저닝하지 않고 일정에 따라 서버리스 기능을 실행함으로써 비용과 관리를 최소한으로 유지

⍢ 서버를 업데이트할 필요가 없고, 유휴 시 비용 지불 또한 불필요하다.
