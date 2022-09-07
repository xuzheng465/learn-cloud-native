MAC Address Length

![image-20220905222608214](assets/image-20220905222608214.png)



![image-20220905224351490](assets/image-20220905224351490.png)





### Ethernet Frame Format

<img src="assets/image-20220905230053870.png" alt="image-20220905230053870" width="200" />



**Preamble**

**SFD** Start Frame Delimiter: signals the end of the preamble

**Dest. MAC**

**Source MAC**

**Length or Type**: 

* size of the payload (if the value is 1500 or less)
* the type of network layer protocol encapsulated (if the value is 1536 or higher)

**Payload**: The Layer 3 Packet containing data to be transmitted （46 - 1,500）

**FCS** Frame Check Sequence: Mathematically determines if the frame was corrupted in transit 



An Enternet header adds 18 bytes (Dest. MAC + Src. MAC + LengthORType + FCS)

![image-20220905230247849](assets/image-20220905230247849.png)



### VLAN Theory

Virtual LAN

An area of a network throughout which a broadcast will propagate

![image-20220905230643600](assets/image-20220905230643600.png)





### STP

Spanning Tree Protocol

Allows a Layer 2 network to have redundant physical connections while preventing traffic from looping through those redundant connections



STP allows a physically redundant topology tobe logically loop-free

