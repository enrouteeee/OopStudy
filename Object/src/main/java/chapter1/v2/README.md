설계를 변경하기 어려운 이유는 Theater가 Audience와 TicketSeller뿐만 아니라 Audience 소유의 Bag과 TicketSeller가 근무하는 TicketOffice까지 마음대로 접근할 수 있기 때문이다.

Audience와 TicketSeller가 직접 Bag과 TicketOffice를 처리하는 자율적인 존재가 되도록 설계하자.

자기 자신의 문제를 스스로 해결하도록 코드를 변경한 것이다.

캡슐화와 응집도

핵심은 객체 내부의 상태를 캡슐화하고 객체 간에 오직 메시지를 통해서만 상호작용하도록 만드는 것이다. Theater는 TicketSeller의 내부에 대해서는 전혀 알지 못한다. 단지 TicketSeller가 메시지를 이해하고 응답할 수 있다는 사실만 알고 있을 뿐이다.