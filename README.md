# Smart-contract-23
```
pragma solidity ^0.8.0;

contract ReputationEvaluator {
    // Specify the contract address
    address public contractAddress = 0xD53128953C501603Ce0E654D357302DefB4df2eD;

    // Mapping of company addresses to reputation data
    mapping (address => ReputationData) public reputationData;

    // Structure to hold reputation data
    struct ReputationData {
        uint256 governanceScore; // Corporate governance history score (1-100)
        uint256 disputeScore; // Involvement in legal disputes score (1-100)
        uint256 reputationRating; // Automatically generated reputation rating (1-100)
        uint256 numRatings; // Number of ratings submitted
        mapping (address => uint256) ratings; // Mapping of user addresses to ratings
    }

    // Event emitted when reputation data is updated
    event ReputationDataUpdated(address indexed companyAddress, uint256 governanceScore, uint256 disputeScore, uint256 reputationRating);

    // Event emitted when a new rating is submitted
    event NewRating(address indexed companyAddress, address indexed userAddress, uint256 rating);

    // Function to update reputation data for a company
    function updateReputationData(address companyAddress, uint256 governanceScore, uint256 disputeScore) public {
        // Check if the caller is the contract owner
        require(msg.sender == contractAddress, "Only the contract owner can update reputation data");

        // Calculate reputation rating based on governance and dispute scores
        uint256 reputationRating = (governanceScore + disputeScore) / 2;

        // Update reputation data
        reputationData[companyAddress].governanceScore = governanceScore;
        reputationData[companyAddress].disputeScore = disputeScore;
        reputationData[companyAddress].reputationRating = reputationRating;

        // Emit event to notify of updated reputation data
        emit ReputationDataUpdated(companyAddress, governanceScore, disputeScore, reputationRating);
    }

    // Function to submit a new rating for a company
    function submitRating(address companyAddress, uint256 rating) public {
        // Check if the user has already submitted a rating
        require(reputationData[companyAddress].ratings[msg.sender] == 0, "User has already submitted a rating");

        // Update the rating and increment the number of ratings
        reputationData[companyAddress].ratings[msg.sender] = rating;
        reputationData[companyAddress].numRatings++;

        // Calculate the new reputation rating
        uint256 newReputationRating = calculateReputationRating(companyAddress);

        // Update the reputation rating
        reputationData[companyAddress].reputationRating = newReputationRating;

        // Emit event to notify of new rating
        emit NewRating(companyAddress, msg.sender, rating);
    }

    // Function to calculate the reputation rating for a company
    function calculateReputationRating(address companyAddress) internal view returns (uint256) {
        // Calculate the average rating
        uint256 sumRatings;
        for (address userAddress in reputationData[companyAddress].ratings) {
            sumRatings += reputationData[companyAddress].ratings[userAddress];
        }
        uint256 averageRating = sumRatings / reputationData[companyAddress].numRatings;

        // Calculate the reputation rating based on the average rating and governance/dispute scores
        uint256 reputationRating = (averageRating + reputationData[companyAddress].governanceScore + reputationData[companyAddress].disputeScore) / 3;

        return reputationRating;
    }

    // Function to get reputation data for a company
    function getReputationData(address companyAddress) public view returns (uint256 governanceScore, uint256 disputeScore, uint256 reputationRating, uint256 numRatings) {
        return (reputationData[companyAddress].governanceScore, reputationData[companyAddress].disputeScore, reputationData[companyAddress].reputationRating, reputationData[companyAddress].numRatings);
    }
}
```
