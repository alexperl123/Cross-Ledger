# Cross-Ledger 

<a href="https://drive.google.com/file/d/1IS0N-dxnh3QQZfbr3SyFvrlJla60lXUa/view?" target="_blank">Cross Ledger architecture</a>
(Please Refer link for Cross ledger Architecture Diagram)


1. Sender constructs an ILP Prepare packet with the receiver's destination account, the agreed-upon condition, and an amount and expiry of their choice. The sender may include additional data, the formatting of which is determined by the higher-level protocol.

2.Sender sends the ILP Prepare packet to a connector over their authenticated communication channel.

3.Connector gets the packet and determines which account to debit based on which communication channel it was received through (each account holder will either have an open connection, such as a WebSocket, or each message will be sent with authentication information, such as in an HTTPS request).

4.Connector checks whether the sender has sufficient balance or credit to send the amount specified in the packet. If so, the connector reduces the sender's available balance by the value of the packet. If not, the connector returns an ILP Reject packet.

5.Connector uses its local routing tables and the destination ILP address from the packet to determine which next hop to forward the packet to. The connector may use an exchange orderbook or any other method it chooses to set its exchange rate between the incoming and outgoing assets.

6.Connector modifies the packet amount and expiry to apply their exchange rate and move the expiry timestamp earlier (for example, each connector may subtract one second from the expiry to give themselves a minimum of one second to pass the Fulfill on in step 10).

7. Connector forwards the packet to the next hop, which may be another connector. All subsequent connectors go through steps 3-7 (treating the previous connector as the sender) until the packet reaches the receiver.

8.Receiver checks the ILP Prepare packet, according to whatever is stipulated by the higher-level protocol (such as checking whether the amount received is above some minimum specified by the sender). Receiver also checks that the Prepare would not put the connector's unsettled balance above the maximum the receiver tolerates (this is the same as the minimum balance limit of the connector with the receiver).

9. To accept the packet, the receiver returns an ILP Fulfill packet, containing the preimage of the condition in the ILP Prepare packet. The receiver sends the Fulfill back through the same communication channel the Prepare packet was received from. The receiver may include additional data in the Fulfill packet, the formatting of which is determined by the higher-level protocol. If the receiver does not want the ILP Prepare packet or it does not pass one of the checks, the receiver returns an ILP Reject packet instead.

10. If the receiver returned an ILP Fulfill, the connector checks that the fulfillment hashes to the original condition and that the receiver returned it before the expiry in the ILP Prepare packet. If the fulfillment is valid, the connector returns the same ILP Fulfill packet to the previous connector (using the same communication channel they originally received the ILP Prepare from) and the connector credits the receiver's account with the amount in the original ILP Prepare packet. If the receiver returned an ILP Reject packet or the Prepare expired before the Fulfill was returned, the connector returns an ILP Reject packet to the previous connector (note that connectors SHOULD return ILP Reject packets when the Prepare expires, even if they have not yet received a response from the next participant). Each connector repeats this step (crediting the account of the participant that returned the Fulfill) until the Fulfill (or Reject) packet reaches the sender.

11.Sender checks that the preimage in the ILP Fulfill packet matches the condition in their Prepare packet and may read any data returned by the receiver. The sender may keep a local record of the total value of packets fulfilled to determine how much they owe their connector.

12.Sender repeats the process, starting from step 1, as many times as is necessary to transfer the desired total amount of value.
