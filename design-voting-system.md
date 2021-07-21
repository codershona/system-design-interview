# Design a Online Voting System


## Introduction
* Online Voting is a web-based voting system that will help manage elections easily and securely.


## Requirement Analysis
### Functional Requirements

* The voters should be able to register via some proper authority.
* The voters should see the list of candidates present in his constituency.
* A voter should be able to cast his vote to a candidate. That voter should cast only 1 vote.


### Non-functional requirements

* Security is the primary non-functional requirement. 
* The system should be highly available.
* The system should be scalable. It should handle large number of voters voting simultaneously.

## API Design
* APIs used by election authority

* VoterDetails register(personDetails)
* addCandidate(candidateDetails)
* addConstituency(constituencyDetails)
* APIs used by Voters

* List<Candidates> getCandidates()
* vote(Candidate)


## Define data model
#### Tables

* Constituency: constituentId(PK), name, numberOfVoters, numberOfCandidates

* Candidate: candidateId(PK), name, constituentId

* Voter: voterId(PK), name, constituentId, candidateId

* Voting data has similar characteristics of financial data.

* Either the voter should be able to successfully cast his vote or it should be a failure. There should be no intermediate state. In case of failures the voter should be allowed to retry immediately. The data should be ACID compliant.

* The voting data should be consistent throughout the system. If we are replicating the data, we do not want any scenario where one database shows Voter-1 has voted Candidate-1 and other database has an old entry for Voter-1 showing he has voted for Candidate-2. We should always have strong consistency. 

* Latency can take a hit over consistency.

* The system should be highly available so that the voter is allowed to vote whenever he/she wants.

* Considering the above requirements we should go for CA(Consistency, Availibility) database which is RDBMS.



## High level design
##### Modes of online voting

* Desktop or Laptop
* Mobile devices
##### Major factors to be considered

* Security & Encryption
* Digital signatures to enable voter authentication and authorization.
* Data integrity and non-repudiation

#### Security & Encryption
###### Which security measures are necessary?

* We can have external and internal firewalls along with a demilitarized zone(DMZ). DMZ is demilitarized zone. The security needs of an online voting is similar to the needs required while designing an online payment system. It should be based on Public key Infrastructure(PKI). We can also go for military grade encryption of the data. Military grade means using the AES-256 encryption in all the communications.

#### Digital signatures
* These digital signatures will be based on some government id example India has a centralized citizenship record of Aadhar.

###### Where is the voter’s digital signature saved?
* The voter can opt for one of the below options

   * On voter’s computer/mobile app
   * On usb token primarily meant to save digital signatures
* The app installed on computer or phone should be able to support upto N number of users where N has a limit of say 5.

<b>Authentication:</b>

* Kerberos Authentication or Active directory for voter identification

##### Two factor authentication:
* For added security concerns we can have a secondary authentication which can be a token generated and sent via email or text message. It can also be a separate software or hard token like rsa securid issued per voter. If using separate software token we have to ensure that it is not present on the same device where we have the digital signature.

#####Biometric security:
* This will require sophisticated equipments to perform fingerprint or retina scanning before casting vote. 
* These equipments are not commercially viable to be provided to each voter or even a household. But this can be present in future when this technology becomes cheap. If the voter has camera selfie phones we can optionally turn on the camera with his consent to perform some additional validation.

##### How can the voter verify if the vote is casted to the correct candidate?
* The voter using the digital signature and 2 factor authentication can request from the app to know the candidate name he has voted.

##### Problem with this approach:
* This can be an issue as someone might force the voter to open the app and show the name of the candidate he has voted. This can lead to privacy issues.

###### Solution:
* Everytime the app is opened it presents the voter with the list of candidates along with a different serial number.
* Example if the voter logs in the app first time. He will see the following list
  * Serial no. 1 – Bill
  * Serial no. 2 – Steve
  * Serial no. 3 – Mark

* If he/she closes the app and logs in again he will see the following list
  * Serial no. 1 – Mark
  * Serial no. 2 – Bill
  * Serial no. 3 – Steve

* Consider he has voted for Steve which has the serial number 3.
* Now if the voter wants to verify his vote is cast to the correct candidate, using his digital signature and 2 factor authentication he can request from the app to know the serial number of the candidate he has voted. 
* The app will return the value as 3.

* We need to save the above data in persistent storage. To accommodate above requirement we can have another table
* Vote: candidateId(PK), serialNumber

* Our system can allow people to cast their ballot several times but only the final vote should be counted.

##### Data Integrity and Non-repudiation
* Data integrity ensures accuracy and consistency of data over its entire life-cycle. No third-party should be allowed to tamper the data.
* Non-repudiation is the assurance, that in this case if the voter has voted he/she cannot deny that fact. There is a trail of his changes saved somewhere.

##### Auditing service
* The audit service will maintain a database to save audit information with INSERT only property.

##### Automated monitoring services
* We can have an automated service which will independently and continuously monitor the entire system by verifying each transaction of Audit and Main DB.

###### What technology can be used for data integrity?
* Answer: Blockchain
* The problem of data integrity is solved using Merkle * Trees structure in blockchain technologies Bitcoin and Ethereum. We can reuse the same concept of Merkel trees to maintain integrity of voter data.

###### What is Merkle Tree?
* It is a tree where every leaf node is a block of data and every non-leaf node is the hash of its children. 
* Here the root node also called as Merkel node acts as a fingerprint to all the transactions. Merkle tree is also known as Hash tree. Generally we use a Binary Merkle tree for blockchain use.

###### How Merkle Tree checks for data integrity?
* Everytime a transaction is performed we save the following metadata in the transaction header.

   *  Hash of previous block
   * Nonce – A number only used once in the entire blockchain process to make each transaction unique.
   * Merkel Root which contains the hash of all the blocks including the current one.

* So as the number of transactions grow the hash of the new transaction is added to the hash of existing transactions essentially making it in the form of a chain. This chain is tamper-proof because if a malicious entity makes modification to any previous block then the Merkel Root will change and all the other users who have valid copies of the block chain will reject his modifications. We can only add new blocks to the chain and cannot modify existing blocks. Addition of new blocks is allowed only if certain criteria is met. Example in case of Bitcoin one of the criteria is that a user has created a nonce.

###### How is Merkle Tree applied in our use case?
* When a voter casts a vote our “Voting Server” sends the hash to the “Blockchain ecosystem” which generates a nonce and adds it to the existing blockchain.

* Since each vote gets added to the blockchain we are essentially storing a trail of changes each person has made. No person can deny that his vote was removed or otherwise changed because only he has the authority of making those changes thus achieving non-repudiation.

###### Automated monitoring services – Updated with blockchain
* We can have an automated service which will independently and continuously monitor the entire system by verifying each transaction of Audit and Main DB and the blockchain.

##### Note:

* I have deliberately kept the below diagrams used in mock interview part of the course, so that it is easy for you to relate. You need to zoom your screen to see them clearly.



## Scale the design
* To make the system always available we need to make sure there are no single points of failure. If one component breaks the system should not break. We can achieve this by having backups of each of these components, so that if one fails other can take over.

* We can also go for horizontal scaling of the system to ensure in case of heavy load our system can spin new instances of the servers and handle them.

* We can also shard the data based on the constituency as voter details are tightly coupled to his\her location.