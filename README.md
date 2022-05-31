# Smartcontract-Walletmoney
// SPDX-License-Identifier: Unlicensed

pragma solidity ^0.8.7;

contract walletmoney {
    // owner wallet
    address owner;

    event LogpeopleFundingReceived(address addr, uint amount, uint contractBalance);

    constructor() {
        owner = msg.sender;
    }

    // define people
    struct people {
        address payable walletAddress;
        string firstName;
        string lastName;
        uint releaseTime;
        uint amount;
        bool canWithdraw;
    }

    people[] public ppl;

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the owner can add people");
        _;
    }

    // add people to contract
    function addpeople(address payable walletAddress, string memory firstName, string memory lastName, uint releaseTime, uint amount, bool canWithdraw) public onlyOwner {
        ppl.push(people(
            walletAddress,
            firstName,
            lastName,
            releaseTime,
            amount,
            canWithdraw
        ));
    }

    function balanceOf() public view returns(uint) {
        return address(this).balance;
    }

    //deposit funds to contract, specifically to a people's account
    function deposit(address walletAddress) payable public {
        addTopplBalance(walletAddress);
    }

    function addTopplBalance(address walletAddress) private {
        for(uint i = 0; i < ppl.length; i++) {
            if(ppl[i].walletAddress == walletAddress) {
                ppl[i].amount += msg.value;
                emit LogKidFundingReceived(walletAddress, msg.value, balanceOf());
            }
        }
    }

    function getIndex(address walletAddress) view private returns(uint) {
        for(uint i = 0; i < ppl.length; i++) {
            if (ppl[i].walletAddress == walletAddress) {
                return i;
            }
        }
        return 999;
    }

    // people checks if able to withdraw
    function availableToWithdraw(address walletAddress) public returns(bool) {
        uint i = getIndex(walletAddress);
        require(block.timestamp > ppl[i].releaseTime, "You cannot withdraw yet");
        if (block.timestamp > ppl[i].releaseTime) {
            ppl[i].canWithdraw = true;
            return true;
        } else {
            return false;
        }
    }

    // withdraw money
    function withdraw(address payable walletAddress) payable public {
        uint i = getIndex(walletAddress);
        require(msg.sender == ppl[i].walletAddress, "You must be the one to withdraw");
        require(ppl[i].canWithdraw == true, "You are not able to withdraw at this time");
        ppl[i].walletAddress.transfer(ppl[i].amount);
    }

}
