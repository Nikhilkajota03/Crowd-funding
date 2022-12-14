# Crowd-funding
//solidity crowd funding project

// SPDX-License-Identifier: GPL-3.0

pragma solidity >= 0.5.0 < 0.9.0;

contract CrowdFunding{
    mapping(address=>uint) public contributors;  //address-->ether  (  Ex = contributors[msg.sender]=100)
    address public manager;
    uint public minimumContribution;
    uint public deadline;
    uint public target;
    uint public raisedAmount;
    uint public noOfContributors;

   struct Request{
        string description;
        address payable recipient;
        uint value;
        bool completed;
        uint noofVoters;
        mapping(address=>bool) voters;
   }
    mapping(uint=>Request) public requests;  // providing the no to the each and every address like giving index
    uint public numRequests;  

    constructor(uint __target, uint _deadline){
        target=__target;
        deadline=block.timestamp+_deadline; //10 sec + 3600sec(60*60)
        minimumContribution=100 wei;
        manager=msg.sender;

    }  

    function sendEth() public payable{
        require(block.timestamp < deadline, "Deadline has passed");
        require(msg.value >=minimumContribution,"Minimum Contribution is not met" );

           if(contributors[msg.sender]==0){  //we have used the if statment to avoid the duplicate increase in the no of contributors
               noOfContributors++;  // we increase the value of the contribution
           }        
           contributors[msg.sender]+=msg.value;  //we have transfer the value of the msg.value to the message.sender
           raisedAmount+=msg.value;  //now we have added the the value of msg.sender to the  raised amount
    }
    function getContractBalance() public view returns(uint){
            return address(this).balance;


    }
    function refund() public{
          require(block.timestamp>deadline && raisedAmount<target, "you are not eligible for refund" );
          require(contributors[msg.sender]>0);
          address payable user=payable(msg.sender); //first we will make it payable so we can send the value to the msg.sender
          user.transfer(contributors[msg.sender]); // whataver the value is in the   contributors[msg.sender]     that value has to be transfered by the user
          contributors[msg.sender]=0; 
    }
    modifier onlyManager(){
        require(msg.sender==manager,"Only manager can call this function");
        _;
     }
     function createRequests(string memory _description, address payable _recipient, uint _value) public onlyManager{
        Request storage newRequest = requests[numRequests]; 
        numRequests++;
        newRequest.description= _description;
        newRequest.recipient= _recipient;
        newRequest.value= _value;
        newRequest.completed= false;
        newRequest.noofVoters=0;

     }

     //for which request no our contributor is voting
     function voteRequest(uint _requestNo) public{
         require(contributors[msg.sender]>0, "You must be contributor");
         Request storage thisRequest=requests[_requestNo];
         require(thisRequest.voters[msg.sender]==false,"you have already voted");
         thisRequest.voters[msg.sender]=true;
         thisRequest.noofVoters++;
     }
     function makePayment(uint _requestNo) public onlyManager{
         require(raisedAmount>=target);
         Request storage thisRequest=requests[_requestNo];
         require(thisRequest.completed==false,"this request has been completed");
         require(thisRequest.noofVoters >noOfContributors/2, "Majority does not support");
         thisRequest.recipient.transfer(thisRequest.value);
         thisRequest.completed=true;
     }
 
}

