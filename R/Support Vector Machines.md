```R
library(e1071)
library(foreach)
library(TTR)
library(quantmod)


#CSV 파일은 추후 올릴 예정입니다
#quantmod에서 제공하는 기능을 이용하여 기술적 지표 데이터 셋을 추가하였습니다.
#Z-score Normalization을 통해 표준화 하였습니다.
#이 코드의 목적은 기존의 SVM을 이용한 주가 예측이 비교적 정확하다는 선행 연구 논문을 검증하고자 했습니다.
#모델을 이용한 결과 , 단순히 SVM을 이용한 주가예측은 동전 던지기 수준보다 약간 높은(55%~60%)정도의 정확도만 보입니다.
#해결할 부분으로는  Crucial한 영향력을 줄 수 있는 변수를 추가해야 하며 개별 주가 종목의 특성을 고려하여 모델을 업그레이드 해야 한다고 판단했습니다.

s <- read.csv("C:/NAVER.csv")
s <- s[1:1964, ]
#obv
#s$obv <- scale(OBV(s$adjusted, s$sigmaDay))

#학습 시키는 데이터 구간이 이동해야 함 (학습 구간이 너무 길면 별로임 한 한달 정도로 해봐야)
sum(is.na(s))

#모멘텀
#s$momentum <- scale(momentum(s[,"close"]))

#지수이동평균
#s$macd <-  scale(MACD(s[,"close"], nFast = 12, nSlow = 26, nSig = 9, maType = "EMA"))


#스프레드
#s$spread <- scale(runMean(s$adjusted, n=5) - runMean(s$adjusted, n=10))

#볼린저 밴드
#s$boll <- scale(BBands(HLC(s), n = 20)[, "pctB"])

#atr
#s$atr <- scale(ATR(scale(HLC(s)), n = 14)[,"atr"])

#atr관련 설명
#http://layhope.tistory.com/241

#aroon
#s$aroon <- scale(aroon(s[, c('high','low')], n = 20))

#chaikinAD
#s$clv <- scale(CLV(s[,c('high','low','close')]))

#RSI
#s$rsi <- scale(RSI(s[,'close'], n=5))

#smi
#s$smi <- scale(SMI(HLC(s), n = 13)[,"SMI"])
#s$stoch <- scale(stoch(s[,c('high','low','close')]))

#chande momentum oscilator
#s$cmo <- scale(CMO(s[,"close"]))

#adx
#s$adx <- scale(ADX(HLC(s), n = 14)[,"ADX"])

#chaikin Vol
#s$chaikin <- scale(chaikinVolatility(s[,c('high','low')]))

#SAR
#s$sar <- scale(SAR(s[, c('high','low')]))

#ADX는 DI의 단점을 보완하기 위하여 이용되는 지표로 일반적으로 DMI를 차트상으로 표시할 때,
#세 개의 선을 이용하여 기술적 분석 지표로 사용한다.
#http://blog.naver.com/PostView.nhn?blogId=zoro8055&logNo=80197779176&parentCategoryNo=&categoryNo=48&viewDate=&isShowPopularPosts=false&from=postView

#CMF
#s$cmf <- scale(CMF(s[ , c('high','low', 'close')], s[,'volume']))

#MFI
#s$mfi <- scale(MFI(s[ , c("high","low","close")], s[,"volume"]))


#ROC 1일전 종가 기준!!
s$rtn <-ROC(s$adjusted)

#MACD 20일
#s$macd <- scale(runMean(s$rtn, n = 10))

#lrClass와 roc 중복으로 인한 제외
s<-subset(s,select = -lrClass)
#log(s$adjusted/s$adjusted[-1])

s <- subset(s, select =-logreturn)
sum(is.na(s))

s$class <- ifelse(s$rtn < 0, 1 ,2)#1%


s$class <- as.factor(s$class) 

#예측을 다음날 분류를 예측해야 하므로 1개 당겨준다.
s$class[1:length(s$class)]<-s$class[-1]

s<-s[28:768, ]


sum(is.na(s))
#불필요 변수 제거 해봄(기준은 단계선택법)
s <- subset(s, select =-volume)
s <- subset(s, select =-betaYearDay)
s <- subset(s, select =-pbr)
s <- subset(s, select =-foreigners)
s <- subset(s, select =-institutional)
s <- subset(s, select =-kodexGBond3Year)
#s <- subset(s, select =-vkospi200)
s <- subset(s, select =-shanghai)
#s <- subset(s, select =-wonPerDollar)
s <- subset(s, select =-wonPerYen)
s <- subset(s, select =-goldFuture)
s <- subset(s, select =-wtiNY)
s <- subset(s, select =-kospi200IT)
s <- subset(s, select =-rtn)
#s <- subset(s, select =-kospi)
#s <- subset(s, select =-sigmaDay60)
#s <- subset(s, select =-dowjones)
s <- subset(s, select =-volumeRTN)
#s <- subset(s, select =-pos_art_rate)
#s <- subset(s, select =-neg_art_rate)
#s <- subset(s, select =-pos_score)
#s <- subset(s, select =-score)

#나머지 변수 scale
#s$pbr <- scale(s$pbr)

#s$institutional <- scale(s$institutional)

s$vkospi200 <- scale(s$vkospi200)

#s$foreigners <- scale(s$foreigners)

#s$shanghai <- scale(s$shanghai)

#s$kospi200IT <- scale(s$kospi200IT)

#s$kodexGBond3Year <- scale(s$kodexGBond3Year)

s$sigmaDay60 <- scale(s$sigmaDay60)

#s$goldFuture <- scale(s$goldFuture)

s$kospi <- scale(s$kospi)

s$dowjones <- as.numeric(s$dowjones)

s$dowjones <- scale(s$dowjones)

#s$volume <- scale(s$volume)

#s$volumeRTN <- scale(s$volumeRTN)

s$wonPerDollar <- scale(s$wonPerDollar)

#s$wtiNY <- scale(s$wtiNY)

#s$score <- scale(s$score)
#s$neg_art_rate <- scale(s$neg_art_rate)



#hynix <- hynix[1:1955,]
#s$lrClass <- as.factor(s$lrClass)
sum(is.na(s))


#n <- as.integer(nrow(s) * 0.1 )
sumAccuracy<-c()
sumDirAcc<-c()
t<-table(c(1,2),c(1,2))-table(c(1,2),c(1,2))

for(i in 1:235){
  trainS <- s[((433+i):(532+i)), ] #네이버는 거래 정지 있음 분할 상장
  testS <- s[(533+i), ]
  
  
  trainS <- as.data.frame(trainS)
  testS <- as.data.frame(testS)
  
  test<- subset(testS, select=-dates)
  
  train<- subset(trainS, select=-dates)
  

  
  train <-subset(train, select =-adjusted)
  test <-subset(test, select =-adjusted)
  
  train <- subset(train, select =-open)
  test <-subset(test, select = -open)
  
  train <- subset(train, select =-high)
  test <- subset(test, select =-high)
  
  train <- subset(train, select =-low)
  test <- subset(test, select =-low)
  
  train <- subset(train, select =-close)
  test <- subset(test, select =-close)
  
  sum(is.na(train))
  sum(is.na(test))
  
 
  
  #바꿔주면서 해보기 
  m1 <-svm(train$class ~., 
           data=train, degree=5, gamma=1, kernel="polynomial", cost=50)  
           
  pred1<-predict(m1, test)
  
  #polynomial의 속도가 가장 빠르다는 선행 연구 :Support Vector Machine에 대한 커널 함수의 성능 분석, 심우성, 성세영, 정차근
  #degree, gamma, cost값을 조정해가면서 판단할 것.
  
  
  
  #}
  test[is.na(test)==TRUE]
  train[is.na(train)==TRUE]
  #train$logreturn<- as.factor(train$logreturn)
  #train
  #str(train)
  #test$logreturn <-as.factor(test$logreturn)
  #str(test)

  
  summary(m1)
  
  length(which(test$class==1))
  length(which(test$class==2))

  
  table(train$class)
  #pred1 <- predict(m1, y)
  #pred1 <- round(pred1)
  
  a <- table(test$class, pred1)
  t<-t+a
  print(a)
  b <- matrix(a, nrow=4, ncol=4)
  
  accuracy <- sum(diag(a)) / sum(a)
  
  #방향
  
  cat("예측 정확도 :", accuracy)
  sumAccuracy[i]<-accuracy
  }



sum(sumAccuracy)/length(sumAccuracy)

a <- table(test$class, pred1)
print(a)
print(t)

#}
#svmFunction(s, 0.94, 10, 5, 10)
#s$class <- as.numeric(s$class)
#model <- lm(s$class ~ s$betaYearDay+s$sigmaDay60+s$pbr+s$institutional+s$foreigners+
              #s$kodexGBond3Year+s$vkospi200+s$wonPerYen+s$wonPerDollar+s$dowjones+
              #s$shanghai+s$goldFuture+s$wtiNY+s$kospi+s$kospi200IT+s$spread+
              #s$atr+s$adx+s$aroon+s$rtn+s)

#s$class <- as.numeric(s$class)
#model <- lm(s$class ~ s$momentum+s$sigmaDay60+s$dowjones+s$kospi+s$mfi+s$cmo+s$volumeRTN)


#summary(model)
#step(model, direction = "both")

##################################################################

```
