//SPDX-Licence-Identifier:GPL-3.0
pragma solidity ^0.8.0;

contract AuctionPlatform {
    struct Auction {
        uint256 auctionID;
        string description;
        uint256 startTime;
        uint256 endTime;
        uint256 minBidValue;
        address auctionOwner;
        bool closed;
    }

    struct Bid {
        uint256 auctionID;
        uint256 bidValue;
        address bidder;
    }

    mapping (uint256 => Auction) public auctions;
    mapping (uint256 => Bid[]) public bids;

    uint256 public auctionCounter;

    function createAuction(string memory _description, uint256 _startTime, uint256 _endTime, uint256 _minBidValue) public returns (uint256) {
        require(_endTime > _startTime, "End time must be greater than start time");
        require(_minBidValue > 0, "Minimum bid value must be greater than 0");
        auctionCounter++;
        auctions[auctionCounter] = Auction(auctionCounter, _description, _startTime, _endTime, _minBidValue, msg.sender, false);
        return auctionCounter;
    }

    function placeBid(uint256 _auctionID, uint256 _bidValue) public payable {
        Auction storage auction = auctions[_auctionID];
        require(block.timestamp >= auction.startTime, "Auction has not started yet");
        require(block.timestamp <= auction.endTime, "Auction has ended");
        require(msg.value >= _bidValue, "Bid value must be greater than or equal to the sent ether");
        require(_bidValue >= auction.minBidValue, "Bid value must be greater than or equal to the minimum bid value");
        Bid[] storage auctionBids = bids[_auctionID];
        auctionBids.push(Bid(_auctionID, _bidValue, msg.sender));
    }

    function getBidList(uint256 _auctionID) public view returns (Bid[] memory) {
        return bids[_auctionID];
    }

    function getAuctionList(address _auctionOwner) public view returns (Auction[] memory) {
        Auction[] memory auctionList = new Auction[](auctionCounter);
        uint256 auctionCount = 0;
        for (uint256 i = 1; i <= auctionCounter; i++) {
            Auction storage auction = auctions[i];
            if (auction.auctionOwner == _auctionOwner) {
                auctionList[auctionCount] = auction;
                auctionCount++;
            }
        }
        assembly {
            mstore(auctionList, auctionCount)
        }
        return auctionList;
    }

    function closeAuction(uint256 _auctionID, uint256 _bidIndex) public {
        Auction storage auction = auctions[_auctionID];
        require(msg.sender == auction.auctionOwner, "Only the auction owner can close the auction");
        require(!auction.closed, "Auction has already been closed");
        Bid[] storage auctionBids = bids[_auctionID];
        require(_bidIndex < auctionBids.length, "Invalid bid index");
        Bid storage winningBid = auctionBids[_bidIndex];
        payable(auction.auctionOwner).transfer(winningBid.bidValue);
        auction.closed = true;
    }

    function getAuctionDetails(uint256 _auctionID) public view returns (uint256, string memory, uint256, uint256, uint256, address, bool) {
        Auction storage auction = auctions[_auctionID];
        return (auction.auctionID, auction.description, auction.startTime, auction.endTime, auction.minBidValue, auction.auctionOwner, auction.closed);
    }
}
