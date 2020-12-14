# Smart Auction
The Auction Smart Contract runs through 4 sequential and dependent phases:

- Bid Phase
- Reveal Phase
- Pay Phase
- Confirm/Abort Phase

And deals with 2 actors:

- Producer
- Participants

### 1] Bid Phase

The producer starts the Auction by deploying the contract, where he/she pays a
deposit value *(which is held by the contract for the entire auction)*, and also
specifies 2 parameters:

- Bid Phase Duration
- Reveal Phase Duration

And from the very first moment of deployment, the bidding phase starts, in
which participants are required to transfer the deposit value to the contract,
alongside with a hash for their bid, concatenated with a secret random string.

***Assumption #1: It is the participant responsibility to choose a long secure random
value for the secret string.***

***Assumption #2: Bidders are allowed only to bid one.***

***Assumption #3: Hashes should be encoded without padding (packed).***

***Assumption #4: Time sent to the contract is in seconds.***

***Assumption #5: Zero is not allowed as a bidding value due the native zero
initialization of the EVM.***

***Assumption #6: Address 0x0 is considered to be a safe placeholder for the winner
address.***

After the bidding duration is over, the contract blocks any further bids, and
starts the revealing phase.

### 2] Reveal Phase

Participants sends their bid values as integers (without transferring it to the
contract yet), as well as the secret random string used in the previous phase,
after which the contract checks the correctness of the hash, and returns the
deposit value to every participant with a correct hash.
This gives an incentive to the participants to reveal their values even though
they might know they lose.

Participants with a wrong hash do not get their deposit back, and that is another
incentive for them to send real values.

Through this phase, the contract keeps track of the highest two bid values, as
well as the address of the highest bidder.

After that, the contract clears the bidding history to avoid double spending by
correct revealers, and to lock the deposits of the wrong revealers.

***Assumption #7: Tie case is handled naively, where the earlier revealer wins.***

### 3] Pay Phase

After reveal phase is over, the winner should pay to the contract an amount of
the second highest bid value, plus the deposit value decided on the beginning
of the bidding phase.

***Assumption #8: The participants should be monitoring the network to know the
winner of the bid, no events are triggered for simplicity.***

***Assumption #9: Winner DoS is not accounted for. An alternative design might be to
return deposits only for participants who lose, so that the winner deposit is kept in
the contract, giving him/her an incentive to pay. This was not implemented though.***

***Assumption #10: All interaction after winner announcement should have taken place
in another contract involving only the producer and winner, but kept in the same
contract for simplicity.***

After the winner pays to the contract, the producer should have an incentive to
send the recording file to the winner (off-chain), as he/she will not receive the
bid value unless the winner confirms the recipient.

The winner then can do one of two things:

### 4-a] Confirm Phase

Where he/she sends an acknowledgment of receiving the correct file. The
winner has an incentive to do so in order to receive his/her deposit back.
In this case, the contract finally pays the bid value (in addition to his/her initial
deposit) to the producer, and the auction ends.

### 4-b] Abort Phase

The winner however can claim that the producer did not send the correct file, or
did not send anything at all. In such case, the contract pays back only the bid
value to the winner, while keeping the deposits on-hold forever.
This should give an incentive to the winner not to lie about receiving the file, as
he/she should loose the deposit value in that case, and also an incentive to the
producer not to send wrong files in order to not loose the deposit.

***Assumption #11: It is the producer’s responsibility to choose the deposit value wisely
so that the winner has an incentive to acknowledge the reception of the correct file.
If the deposit value is not high enough, the winner might decide to sacrifice the
deposit value to get away with the file and the bid value.***


## Test Case:

A participant might want to bid by the value 12345, and for that he/she
chooses the secret word “Heythere”.

At the bid phase he/she will send the hash:
***0x300aeef702d3d96812b9f920ec513609477c7a42ef5871dffd1c626a77c9a***

which is the 256 bit Keccak hash for the encoded value:
***0x0394865797468657265***

which itself is the encoded packed value for the parameters:
***12345***, ***Heythere*** (without quotes).

In the reveal phase then, the participant would show out the parameters, so
that the hash can be recalculated by the contract.
