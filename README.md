# Blockchain Program Written in Bash


## Digital Currency and Blockchain Overview
A blockchain is a series of files arranged so that when a new file is inserted, it both verifies the previous file and renders it unchangeable. The files (blocks) in the chain are safeguarded from alteration since all files are linked to one another. This proves to be extremely useful for both record retention in a records management system and a trust-less decentralized digital currency.

A digital currency can use a blockchain to create an immutable list of transactions. This means that, under the correct circumstances, once a transaction is saved to a block it cannot be altered or deleted without breaking the chain.

When people need to exchange goods and services they most often use physical currency. Trading a dollar bill for a soda is much easier than trying to figure out for how long a painter would paint the store owner’s house in exchange for the soda if the store owner needed painting at all.  As long as both the painter and the store owner agree that the dollar bill is worth something, they can exchange it instead of bartering.

Instead of physically exchanging something during a monetary transaction, however, participants can simply keep track on a ledger: a running list of financial transactions.  For example, a transaction such as “Alice sends Bob 100 dollars” can be written into a ledger that both Bob and Alice trust.  The transaction information can then be used to calculate how much money both Alice and Bob have at any moment.

If the ledger is shared amongst all users then anyone should be able to submit transactions and calculate how much money anyone has at any moment.  A public ledger setup as a blockchain has the added capability of keeping past transactions secure as well as protecting users from future fraud.  Users are also required to use a public key / private key pair as their account address which they use to encrypt transactions. This encryption prevents participants from writing false transactions to the ledger.

Transactions are grouped into blocks before being added to the blockchain. The frequency at which they are added depends on other users verifying the block through a process called mining.  Mining a block consists of adding a random number (called a nonce) to the block so that when it is hashed the resulting hash starts with a certain number of zeros.  This process is extremely difficult and time-consuming but once the random number has been found, reversing the process to check the answer is simple and quick.  The miner who found the random number that produces a qualifying hash is then rewarded with a predetermined amount of digital currency as well as fees paid by users.  The transaction that pays the miner is listed first in the next block in the ledger.

Mining a block and being financially rewarded is one of the things that give value to the currency. Forcing users to work for the monetary reward makes it so that others that are not mining appreciate their effort and value the currency in circulation.  The reward is automatically given to the miner and it is created as if it were mined out of the Earth.  It was not owned by anyone before the miner and is a finite resource. The financial reward given to miners decreases over time and will eventually reach zero.

To better keep track of the ledger as well as ensuring security, a cryptocurrency such as Bitcoin can use change transactions.  When a user sends another user coins, he or she will send themselves their remaining balance to a brand new address.  The new address is encrypted and unrelated to the address before it.  For example, if Alice has a total of 300 coins and she wants to send Bob 100, she would send Bob 100 coins and send 200 coins to a brand new public/private key pair address.  She would generate this new address when she sends the coins to Bob.  Alice would be the only owner of the new private key.

## Genesis
Blocks in the blockchain are assembled with the hash of the previous block.  Starting a blockchain requires a block that does not rely on a previous block.  This block is called the genesis block. The genesis block is special in that its hash is not calculated using the hash of another block.

bashCoin has a function called mineGenesis() that mines a file without checking the hash of another file.  This function is called at the terminal with the command:
<pre>
./bashCoin.sh minegen <file>
</pre>

The function first checks to see if a genesis block exists so that it can abort before overwriting it:
<pre>
[[ -f 1.blk.solved ]] && echo "A mined Genesis block already exists in this folder!" && exit 1
</pre>
Next, the function will enter a loop, adding a nonce to the file and checking if its md5 hash starts with the same number of $ZEROS.  $DIFFZEROS is the difficulty variable that is calculated by bashCoin.
<pre>
while [ $ZEROS != $DIFFZEROS ]
 do
 let "NONCE += 1"
 HASH=$(printf -- "`cat $1`\n\n## Nonce: #################################################################################\n$NONCE\n" | md5sum)
 HASH=$(echo $HASH | cut -d" " -f1)
 echo $HASH
 ZEROS=$(echo $HASH | cut -c1-$DIFF)
done
</pre>
When the nonce is found that produces a hash that begins with the correct number of zeros bashCoin will display the results and, more importantly, build the next block in the chain.  The next block includes the hash of the genesis block.

<pre>
echo "Success!"
echo "Nonce:    " $NONCE
echo "Hash:     " $HASH
printf -- "`cat $1`\n\n## Nonce: #################################################################################\n$NONCE\n" > 1.blk.solved
printf "## Previous Block Hash: ###################################################################\n" >> 2.blk
printf "$HASH\n\n" >> 2.blk
</pre>
bashCoin starts the blockchain with the files “1.blk.solved” and “2.blk”.  The file that was used as the genesis is not deleted from the system but it is no longer needed for the blockchain.  When subsequent blocks are solved the “.solved” file remains but the “.blk” is deleted.

While Bitcoin rewards miners with coins, bashCoin currently does not.  Because of this, it is a good idea to include a large transaction in the genesis block before mining so that at least one participant has wealth.  bashCoin uses a specific schema for transactional data.  The following two lines can be included in the genesis file for ease of use.

<pre>
tx1:user1:user1:1000000

tx2:user1:user1:1000000
</pre>
When the genesis block is mined, user1 will “own” 1 million bashCoins.  user1 can then send up to 1 Mil coins to any other address.

## Sending Bash Coins
After the first two transactions are included in the genesis block and it is mined, user1 can send coins to another address.  Addresses in bashCoin can be thought of as accounts that don’t need to be set up prior to receiving coins. In Bitcoin, addresses are essentially public/private key pairs.  They must be calculated before they are used. After an address in Bitcoin receives payment the private key is the only thing that allows a user to spend the balance. In bashCoin, however, any user can send transactions from any account.  There is currently no security on bashCoin addresses.

The following command allows a user to send coins in bashCoin.
<pre>
./bashCoin.sh send <numOfCoins> <toAddress> <fromAddress>
</pre>
The <toAddress> can be any string of letters and numbers as in “user2”.  bashCoin will go through the blockchain and calculate the balance of <fromAddress> so it must be associated with a positive balance for the transaction to work.  Use the following example to send the first transaction:
<pre>
./bashCoin send 10 user2 user1
</pre>
The send() function in bashCoin first assigns the arguments to variables.  (The arguments are first shifted before the send() function is called.)
<pre>
SENDER=$3
RECEIVER=$2
AMOUNT=$1
</pre>
After checking the current block to make sure there is a valid and live chain, send() greps the current block for the latest transaction number.  If the current block does not have transactions listed it will  look to the previous block for the latest transaction number.  If the last transaction number cannot be found then the script exits.
<pre>
if [[ `cat $CURRENTBLOCK | grep "tx[0-9]*:"` ]]
 then
 #echo "block has transactions"
 # Get next tx number
 LASTTRAN=`cat $CURRENTBLOCK | tail -1 | cut -d":" -f 1 | sed 's/tx//g' | sed 's/^0*//g'`
 NEXTTRAN=`echo $LASTTRAN + 1 | bc`
 #echo Next transaction number $NEXTTRAN
 else
 #echo "block is void of transactions!"
 # Get next tx number from previous block
 LASTTRAN=`cat $PREVIOUSBLOCK | grep tx | tail -1 | cut -d":" -f 1 | sed 's/tx//g' | sed 's/^0*//g'`
 NEXTTRAN=`echo $LASTTRAN + 1 | bc`
 #echo Next transaction number $NEXTTRAN
fi

        [[ -z $LASTTRAN ]] && echo "ERROR: Couldn't find last transaction number" && exit 1
</pre>
When the next transaction number is known, bashCoin will check the balance of the sending address.  checkAccountBalance() is explained below.

<pre>
checkAccountBal $3

echo "Amount to send: $AMOUNT"

[[ $AMOUNT -gt $TOTAL ]] && echo "Insufficient Funds!" && exit 1
If the address has a balance greater than the sending amount, a transaction is written to the current block.

echo "Success! Sent $1 bCN to $2"
echo tx$NEXTTRAN:$SENDER:$RECEIVER:$AMOUNT >> $CURRENTBLOCK
CHANGETRAN=`echo $NEXTTRAN+1 | bc`
CHANGE=`echo $TOTAL-$AMOUNT | bc`
echo tx$CHANGETRAN:$SENDER:$SENDER:$CHANGE >> $CURRENTBLOCK
</pre>

## Checking Balance
bashCoin checks the balance of an address by reverse searching through the blockchain for the last time that address sent itself change.  The last change transaction in the blockchain for an address is the current balance plus any receiving transactions after it.  After bashCoin knows which block contains the users last change transaction, it can search the remaining blocks for receiving transactions.  For instance, if Alice sent Bob 100 coins and sent herself her remaining 200 coins she would end up with 200 coins.  bashCoin finds this last transaction first and then searches forward in time for any other transactions to Alice from other users.  If Bob sent Alice 10 coins after Alice sent him 100, Alice would own 210 coins.

<pre>
checkAccountBal () {

ACCTNUM=$1
# Get the value of the last change transaction (the last time a user sent money back to themselves) if it exists
for i in `ls *.blk* -1r`; do
LASTCHANGEBLK=`grep -l -m 1 .*:$ACCTNUM:$ACCTNUM:.* $i`
[[ $LASTCHANGEBLK ]] && break
done

# If there was a change transaction get the value, if not print "Account has never spent money."
[[ $LASTCHANGEBLK ]] && LASTCHANGE=`grep $ACCTNUM:$ACCTNUM $LASTCHANGEBLK | tail -1` || echo "Account has never spent bashCoin."

# If account has never spent money then set LASTCHANGE to zero, if it has, separate out the tx number and value of that transaction
[[ $LASTCHANGE ]] && LASTCHANGETX=`echo $LASTCHANGE | cut -d":" -f 1 | sed 's/tx//'` && LASTCHANGE=`echo $LASTCHANGE | cut -d":" -f 4` || LASTCHANGE=0

# Get all of the receiving transactions after last change in the last change block and add them together
if [[ $LASTCHANGEBLK ]]
then
RECAFTERCHANGE=`sed "1,/tx$LASTCHANGETX/d" $LASTCHANGEBLK | grep .*:.*:$ACCTNUM:.* | cut -d":" -f4 | paste -sd+ | bc`
[[ $RECAFTERCHANGE ]] || RECAFTERCHANGE=0
#echo "Received after last change tx (same blk) = $RECAFTERCHANGE"
else
LASTCHANGEBLK=1.blk.solved
RECAFTERCHANGE=`grep .*:.*:$ACCTNUM:.* $LASTCHANGEBLK | cut -d":" -f4 | paste -sd+ | bc`
[[ $RECAFTERCHANGE ]] || RECAFTERCHANGE=0
#echo "Received after last change tx (same blk)(genesis) = $RECAFTERCHANGE"
fi


#echo $LASTCHANGEBLK
# Get all the receiving transactions after last change block and add them together
SUM=0
if [[ $LASTCHANGEBLK == 1.blk.solved ]]
then
for i in `ls -1 *.blk*`; do
[[ `grep .*:.*:$ACCTNUM:.* $i` ]] && SUM=$SUM+`grep .*:.*[^$ACCTNUM]*:$ACCTNUM:.* $i | cut -d":" -f4 | paste -sd+ | bc` || SUM=$SUM+0
done
else
for i in `ls -1 *.blk* | sed "1,/$LASTCHANGEBLK/d"`; do
[[ `grep .*:.*:$ACCTNUM:.* $i` ]] && SUM=$SUM+`grep .*:.*[^$ACCTNUM]*:$ACCTNUM:.* $i | cut -d":" -f4 | paste -sd+ | bc` || SUM=$SUM+0
done
fi


SUM=`echo $SUM | bc`
[[ $SUM ]] || SUM=0
#echo "Received after last change tx (remaining blk's)= $SUM"

# Add last change value and received money since then
TOTAL=`echo $LASTCHANGE+$RECAFTERCHANGE+$SUM`
TOTAL=`echo $LASTCHANGE+$RECAFTERCHANGE+$SUM | bc`
echo "Current Balance for $ACCTNUM:             $TOTAL"

}
</pre>

## Validation
The only reason we can trust transactions in this system is because we can validate every transaction from the beginning.  Every transaction ever recorded is grouped together in blocks and hashed.  Each block includes the hash of the block before it.  If a nefarious user tried to change a transaction in a previous block, the hash chain would break.  The hash of that changed block would not match the hash that was listed in the next block when it was mined.

bashCoins’s validation function is named validate().  This function first checks to see if there is more than one block in the chain.
<pre>
NUMSOLVEDFILES=`ls -1 *.solved | wc -l`
(( $NUMSOLVEDFILES < 2 )) && echo "Blockchain must be greater than 1 block to validate" && exit 1
</pre>

Next, it enters a loop that hashes a block, inserts the result into the next block, and hashes the result.  As long as the hashes continue to match, it will continue until the end of the chain.  If the function finds a hash mismatch it will stop and warn the user from trusting any block after the block with the bad hash.
<pre>
for i in `ls -1 *.solved | tail -n +2`; do
h=`ls -1 | grep solved$ | grep -B 1 $i | head -1`
j=`ls -1 | egrep "solved$|blk$" | grep -A 1 $i | grep -v $i | tail -1`
PREVHASH=`md5sum $h | cut -d" " -f1`
CALCHASH=`sed "2c$PREVHASH" $i | md5sum | cut -d" " -f1`
REPORTEDHASH=`sed -n '2p' $j`
echo "Checking $i"
echo "Previous Hash: $PREVHASH"
#echo "Calculated Hash: $CALCHASH  Reported Hash: $REPORTEDHASH"
[[ $CALCHASH != $REPORTEDHASH ]] && echo "Hash mismatch!  $i has changed!  Do not trust any block after and including $i" && exit 1 || echo "Hashes match! $i is a good block."
done
echo "Blockchain validated.  Unbroken Chain."
</pre>

## Mining
Mining in bashCoin is almost the same process as mining the genesis block.  The main difference is that mine() will search for the blocks in the current directory and figure out which one needs to be mined.  Also, the validate() function is called before mine() to first verify that the blockchain is unbroken.  Mining the genesis block does not.

<pre>
mine() {

while [ $ZEROS != $DIFFZEROS ]
do
 # increase the nonce by one
 let "NONCE += 1"
 # do the hashing
 HASH=$(printf "`cat $CURRENTBLOCK.wip`\n\n## Nonce: #################################################################################\n$NONCE\n" | md5sum)
 HASH=$(echo $HASH | cut -d" " -f1)
 # print the hash to the screen because it looks cool
 echo $HASH
 # cut the leading zeros off the hash
 ZEROS=$(echo $HASH | cut -c1-$DIFF)
done
echo "Success!"
echo "Nonce:    " $NONCE
echo "Hash:      " $HASH
printf "`cat $CURRENTBLOCK.wip`\n\n## Nonce: #################################################################################\n$NONCE\n" > $CURRENTBLOCK.solved
rm -f $CURRENTBLOCK.wip
rm -f $CURRENTBLOCK
# Setup the next block.  Add previous hash first
printf "## Previous Block Hash: ###################################################################\n" >> $NEXTBLOCK
printf "$HASH\n\n" >> $NEXTBLOCK

}
</pre>
The mine() function automatically creates the next block and includes the hash for the freshly mined block at the top of the file.

Summary
The simple concepts within Bitcoin solve several complex problems.  Namely, safeguarding wealth with strong encryption and recording an immutable, anonymous, and public ledger can be easily replicated with a bash script.  See below for the current version on bashCoin.  Copy and paste into a Linux terminal to use.
