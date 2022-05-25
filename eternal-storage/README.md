Contract EternalStorage is used for storing values
Contract Ballot is having logic

This is a simple voting Smart Contract. You call vote() and increase a number - pretty basic business logic.

We deploy EternalStorage first & then Ballot(give address of EternalStorage contract). 
Let's say we found a bug, because everyone can vote as many times as they want. We fix it and re-deploy only the Ballot Smart Contract (neglecting that the old version still runs and that there is no way to stop it without extra code).
Replace everything with the following code.

library ballotLib {

    function getNumberOfVotes(address _eternalStorage) public view returns (uint256)  {
        return EternalStorage(_eternalStorage).getUIntValue(keccak256('votes'));
    }

    function getUserHasVoted(address _eternalStorage) public view returns(bool) {
        return EternalStorage(_eternalStorage).getBooleanValue(keccak256(abi.encodePacked("voted",msg.sender)));
    }

    function setUserHasVoted(address _eternalStorage) public {
        EternalStorage(_eternalStorage).setBooleanValue(keccak256(abi.encodePacked("voted",msg.sender)), true);
    }

    function setVoteCount(address _eternalStorage, uint _voteCount) public {
        EternalStorage(_eternalStorage).setUIntValue(keccak256('votes'), _voteCount);
    }
}

contract Ballot {
    using ballotLib for address;
    address eternalStorage;

    constructor(address _eternalStorage) {
        eternalStorage = _eternalStorage;
    }

    function getNumberOfVotes() public view returns(uint) {
        return eternalStorage.getNumberOfVotes();
    }

    function vote() public {
        require(eternalStorage.getUserHasVoted() == false, "ERR_USER_ALREADY_VOTED");
        eternalStorage.setUserHasVoted();
        eternalStorage.setVoteCount(eternalStorage.getNumberOfVotes() + 1);

    }
}

You see, only the Library changed. The Storage is exactly the same as before. But how to deploy the update?

Re-Deploy the "Ballot" Smart Contract and give it the address of the Storage Contract. That's all.

The Storage Contract hasn't changed at all, we don't even need to redeploy it. Just use the one that already exists! You see then that you can vote only one now.
