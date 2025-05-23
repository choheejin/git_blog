# 데이터 정합성을 위한 실시간 동기화 로직 작성

```java
    // TODO: 블록 마지막 sync가 겹칠 수 있으니, sync 테이블 추가 필요
    @Scheduled(fixedRate = 60000)
    public void realTimeSync() {
        log.info("[BatchService] 실시간 동기화 진행: {}", LocalDateTime.now());
        List<BigInteger> last50Blocks = transactionService.getLast50BlockNumbers();

        log.debug("[BatchService] 마지막 블록 번호: {}", last50Blocks.getLast());

        for(BigInteger block : last50Blocks) {
            List<GifticonNFT.NFTPurchasedEventResponse> purchasedEventResponseList = transactionService.getPurchaseEventsByBlockNumber(block);
            // TODO: 같은 블록을 처리하게 되는 것은 아닌지 고민해보기 
            purchasedEventResponseList.parallelStream().forEach(response -> {
                try {
                    if(articleHistoryRepository.findByTxHash(response.log.getTransactionHash()).isEmpty()) {
                        log.debug("[BatchService] DB에 저장되지 않은 Hash 값: {}", response.log.getTransactionHash());
                        articleHistoryRepository.save(
                                ArticleHistory.builder()
                                        .articleId(getArticleId(response))
                                        .createdAt(TimeUtil.convertTimestampToLocalTime(response.transactionTime))
                                        .historyType(ContractType.PURCHASE.getType())
                                        .userId(getUserId(response))
                                        .txHash(response.log.getTransactionHash())
                                        .build()
                        );
                    }
                } catch (Exception e) {
                    log.error("Failed to process txHash: {}", response.log.getTransactionHash(), e);
                }
            });
        }
    }
```

// TODO: parallelStream()에 대해 개념적 공부하고 작성하기
