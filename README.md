// A simple auction contract
contract Auction {
    address public owner;
    uint public auctionEndTime;
    uint public highestBid;
    address public highestBidder;
    mapping(address => uint) public bids;

    constructor(uint _durationMinutes) {
        owner = msg.sender;
        auctionEndTime = block.timestamp + (_durationMinutes * 1 minutes);
    }

    function placeBid() public payable {
        require(block.timestamp < auctionEndTime, "Auction has ended.");
        require(msg.value > highestBid, "Bid must be higher than current highest bid.");

        if (highestBid != 0) {
            bids[highestBidder] += highestBid;
        }
        
        highestBid = msg.value;
        highestBidder = msg.sender;
    }

    function withdraw() public {
        require(msg.sender != highestBidder, "Highest bidder cannot withdraw.");
        require(bids[msg.sender] > 0, "No funds available for withdrawal.");

        uint amount = bids[msg.sender];
        bids[msg.sender] = 0;
        
        (bool success, ) = payable(msg.sender).call{value: amount}("");
        require(success, "Withdrawal failed.");
    }

    function endAuction() public {
        require(msg.sender == owner, "Only owner can end auction.");
        require(block.timestamp >= auctionEndTime, "Auction has not ended yet.");
        
        (bool success, ) = payable(owner).call{value: highestBid}("");
        require(success, "Transfer of funds to owner failed.");
        
        highestBidder = address(0);
        highestBid = 0;
    }
}
