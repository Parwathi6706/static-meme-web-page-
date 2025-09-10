// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title StaticMemeWebPage
 * @dev A decentralized meme platform smart contract
 * @author Static Meme Web Page Team
 */
contract StaticMemeWebPage {
    
    // Struct to represent a meme
    struct Meme {
        uint256 id;
        string title;
        string imageUrl;
        string description;
        address creator;
        uint256 timestamp;
        uint256 votes;
        uint256 tips;
        bool isActive;
    }
    
    // State variables
    mapping(uint256 => Meme) public memes;
    mapping(address => uint256[]) public userMemes;
    mapping(uint256 => mapping(address => bool)) public hasVoted;
    mapping(uint256 => mapping(address => uint256)) public userTips;
    
    uint256 public memeCounter;
    uint256 public totalTips;
    address public owner;
    
    // Events
    event MemeCreated(uint256 indexed memeId, string title, address indexed creator);
    event MemeVoted(uint256 indexed memeId, address indexed voter);
    event MemeTipped(uint256 indexed memeId, address indexed tipper, uint256 amount);
    event MemeStatusChanged(uint256 indexed memeId, bool isActive);
    
    // Modifiers
    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner can call this function");
        _;
    }
    
    modifier memeExists(uint256 _memeId) {
        require(_memeId > 0 && _memeId <= memeCounter, "Meme does not exist");
        _;
    }
    
    modifier memeIsActive(uint256 _memeId) {
        require(memes[_memeId].isActive, "Meme is not active");
        _;
    }
    
    constructor() {
        owner = msg.sender;
        memeCounter = 0;
        totalTips = 0;
    }
    
    /**
     * @dev Core Function 1: Create a new meme
     * @param _title The title of the meme
     * @param _imageUrl The URL of the meme image
     * @param _description The description of the meme
     */
    function createMeme(
        string memory _title, 
        string memory _imageUrl, 
        string memory _description
    ) public {
        require(bytes(_title).length > 0, "Title cannot be empty");
        require(bytes(_imageUrl).length > 0, "Image URL cannot be empty");
        
        memeCounter++;
        
        memes[memeCounter] = Meme({
            id: memeCounter,
            title: _title,
            imageUrl: _imageUrl,
            description: _description,
            creator: msg.sender,
            timestamp: block.timestamp,
            votes: 0,
            tips: 0,
            isActive: true
        });
        
        userMemes[msg.sender].push(memeCounter);
        
        emit MemeCreated(memeCounter, _title, msg.sender);
    }
    
    /**
     * @dev Core Function 2: Vote for a meme
     * @param _memeId The ID of the meme to vote for
     */
    function voteForMeme(uint256 _memeId) 
        public 
        memeExists(_memeId) 
        memeIsActive(_memeId) 
    {
        require(!hasVoted[_memeId][msg.sender], "You have already voted for this meme");
        require(memes[_memeId].creator != msg.sender, "Cannot vote for your own meme");
        
        hasVoted[_memeId][msg.sender] = true;
        memes[_memeId].votes++;
        
        emit MemeVoted(_memeId, msg.sender);
    }
    
    /**
     * @dev Core Function 3: Tip a meme creator with ETH
     * @param _memeId The ID of the meme to tip
     */
    function tipMeme(uint256 _memeId) 
        public 
        payable 
        memeExists(_memeId) 
        memeIsActive(_memeId) 
    {
        require(msg.value > 0, "Tip amount must be greater than 0");
        require(memes[_memeId].creator != msg.sender, "Cannot tip your own meme");
        
        address creator = memes[_memeId].creator;
        
        // Transfer tip to meme creator
        (bool success, ) = payable(creator).call{value: msg.value}("");
        require(success, "Tip transfer failed");
        
        // Update tracking variables
        memes[_memeId].tips += msg.value;
        userTips[_memeId][msg.sender] += msg.value;
        totalTips += msg.value;
        
        emit MemeTipped(_memeId, msg.sender, msg.value);
    }
    
    // View functions
    function getMeme(uint256 _memeId) 
        public 
        view 
        memeExists(_memeId) 
        returns (Meme memory) 
    {
        return memes[_memeId];
    }
    
    function getUserMemes(address _user) 
        public 
        view 
        returns (uint256[] memory) 
    {
        return userMemes[_user];
    }
    
    function getTotalMemes() public view returns (uint256) {
        return memeCounter;
    }
    
    function getMemeVotes(uint256 _memeId) 
        public 
        view 
        memeExists(_memeId) 
        returns (uint256) 
    {
        return memes[_memeId].votes;
    }
    
    function getMemeTips(uint256 _memeId) 
        public 
        view 
        memeExists(_memeId) 
        returns (uint256) 
    {
        return memes[_memeId].tips;
    }
    
    function hasUserVoted(uint256 _memeId, address _user) 
        public 
        view 
        memeExists(_memeId) 
        returns (bool) 
    {
        return hasVoted[_memeId][_user];
    }
    
    function getUserTipAmount(uint256 _memeId, address _user) 
        public 
        view 
        memeExists(_memeId) 
        returns (uint256) 
    {
        return userTips[_memeId][_user];
    }
    
    // Owner functions
    function toggleMemeStatus(uint256 _memeId) 
        public 
        onlyOwner 
        memeExists(_memeId) 
    {
        memes[_memeId].isActive = !memes[_memeId].isActive;
        emit MemeStatusChanged(_memeId, memes[_memeId].isActive);
    }
    
    function getContractBalance() public view returns (uint256) {
        return address(this).balance;
    }
    
    // Emergency function to withdraw contract balance (if any ETH gets stuck)
    function emergencyWithdraw() public onlyOwner {
        require(address(this).balance > 0, "No balance to withdraw");
        (bool success, ) = payable(owner).call{value: address(this).balance}("");
        require(success, "Withdrawal failed");
    }
}
