왜 요청하다가 끝나버려? map으로 응답 들어오기 전에?
    **public** **func** checkPrepaidPaymentIssuingStatus(cardId: Int, cardName: String) -> Observable<Mutation> {
        **let** requestPrePayStatusUseCase = PaymentDIFactoryContainer.shared.usecase.makeFetchPrePayStatus()
        **return** requestPrePayStatusUseCase
            .execute(input: FetchPrePayStatusInput(cardId: cardId))
            .subscribe(on: ConcurrentDispatchQueueScheduler(qos: .background))
            .observe(on: MainScheduler.instance)
            .map { $0.infoModel.isCardComplete }
            .map { isComplete **in**
                Mutation.updatePrepaidPaymentIssuingStatus(cardName, !isComplete)
            }
            .catch { _ **in**
                **return** Observable.just(Mutation.updatePrepaidPaymentIssuingStatus(cardName, **false**))
            }
    }
  
  
 **public** **class** FetchPrePayStatusUseCase: FetchPrePayStatusUseCaseProtocol {
    **public** **let** basePaymentRepository: BasePaymentRepositoryProtocol
    **public** **init**(basePaymentRepository: BasePaymentRepositoryProtocol) {
        **self**.basePaymentRepository = basePaymentRepository
    }
    **public** **func** execute(input: FetchPrePayStatusInput) -> Observable<FetchPrePayStatusOutput> {
        **let** config = PaymoneyConfig(cardID: input.cardId, cardName: "", plateImageUrl: "" , payAmount: "")
        **return** basePaymentRepository.fetchPaymoneyCardInfo(config)
            .map {
                FetchPrePayStatusOutput(infoModel: $0)
            }
            .catch { _ **in**
                **throw** PrePayBalanceUseCaseError.apiError
            }
    }
}
