// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract GRAVAX {
    address public owner;
    uint256 public totalSupply;
    mapping(address => uint256) public balances;
    
    struct ConservationActivity {
        address initiator;
        string description;
        uint256 timestamp;
        bool verified;
        uint256 rewardAmount;
    }
    
    ConservationActivity[] public activities;
    mapping(address => uint256) public lastSubmitted;
    
    event ActivitySubmitted(address indexed initiator, string description, uint256 timestamp);
    event ActivityVerified(uint256 indexed activityId, address indexed initiator, uint256 reward);
    event TokensIssued(address indexed recipient, uint256 amount);
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this function");
        _;
    }
    
    constructor() {
        owner = msg.sender;
        totalSupply = 0;
    }
    
    function submitActivity(string memory _description) public {
        require(block.timestamp - lastSubmitted[msg.sender] > 1 days, "Only one submission per day allowed");
        
        activities.push(ConservationActivity({
            initiator: msg.sender,
            description: _description,
            timestamp: block.timestamp,
            verified: false,
            rewardAmount: 0
        }));
        
        lastSubmitted[msg.sender] = block.timestamp;
        emit ActivitySubmitted(msg.sender, _description, block.timestamp);
    }
    
    function verifyActivity(uint256 _activityId, uint256 _rewardAmount) public onlyOwner {
        require(_activityId < activities.length, "Invalid activity ID");
        require(!activities[_activityId].verified, "Activity already verified");
        
        activities[_activityId].verified = true;
        activities[_activityId].rewardAmount = _rewardAmount;
        
        balances[activities[_activityId].initiator] += _rewardAmount;
        totalSupply += _rewardAmount;
        
        emit ActivityVerified(_activityId, activities[_activityId].initiator, _rewardAmount);
        emit TokensIssued(activities[_activityId].initiator, _rewardAmount);
    }
    
    function getActivity(uint256 _activityId) public view returns (ConservationActivity memory) {
        require(_activityId < activities.length, "Invalid activity ID");
        return activities[_activityId];
    }
}
