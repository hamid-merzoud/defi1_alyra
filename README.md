# defi1_alyra

pragma solidity ^0.8.9;

import "./Ownable.sol";

contract Voting is Ownable {
    
    uint public winningProposalId;
    uint8 idProposal;
    
    event VoterRegistered(address voterAddress); 
    event WorkflowStatusChange(WorkflowStatus previousStatus, WorkflowStatus newStatus);
    event ProposalRegistered(uint proposalId);
    event Voted (address voter, uint proposalId);
    event VotesTallied();
    
    struct Voter {
        address _address;
        bool isRegistered;
        bool hasVoted;
        uint votedProposalId;
    }
    
    struct Proposal {
        uint8 id;
        address owner;
        string description;
        uint voteCount;
    }
    
    
    mapping(address => Voter) public whitelist;
    mapping(address => Voter) public voters;
    mapping(uint => Proposal) public proposals;
    
    
    enum WorkflowStatus {
        RegisteringVoters,
        ProposalsRegistrationStarted,
        ProposalsRegistrationEnded,
        VotingSessionStarted,
        VotingSessionEnded,
        VotesTallied
    }
    
    WorkflowStatus status;
    
    
     function getWinningProposal() public view returns(Proposal memory proposal) {
        return proposals[winningProposalId];
    }
    
    
    //ajout d'adresse a la whitelist
    function addVoter(address _address) public onlyOwner {
        require(status == WorkflowStatus.RegisteringVoters);
        Voter memory newVoter = Voter(_address, true, false, 0);
        whitelist[_address] = newVoter;
    }
    
     //verifier le gas
    function deleteVoter(address _address) public onlyOwner {
      delete whitelist[_address];
    }
    
      // Verification de la presence de l'address dans whitelist
    modifier whiteListed() {
      require(whitelist[msg.sender].isRegistered == true);
      _;
    }
    
    // _description est le nom de la proposition
    function addProposal(string memory _description) public whiteListed {
      //obligation d'être dans le workflow correspondant 
      require(status == WorkflowStatus.ProposalsRegistrationStarted);
      Proposal memory newProposal = Proposal(idProposal, msg.sender, _description, 0);
      proposals[idProposal] = newProposal;
      emit ProposalRegistered(idProposal);
      idProposal++;
    }
    
    function deleteProposal(uint _id) public whiteListed {
      require(proposals[_id].owner == msg.sender);
      delete proposals[_id];
    }

    function vote(uint _id) public whiteListed {
      require(status == WorkflowStatus.VotingSessionStarted);
      require(whitelist[msg.sender].hasVoted == false);
      whitelist[msg.sender].hasVoted = true;
      proposals[_id].voteCount++;
      emit Voted(msg.sender, _id);
    }

    function count() public onlyOwner {
        require(status == WorkflowStatus.VotingSessionEnded);
        uint8 id;
        uint highestCount;

        for (uint i = 0; i < idProposal; i++) {
          if (highestCount < proposals[i].voteCount) {
             id = proposals[i].id;
          }
     // à voir les cas particuliers où plusieurs gagnants
        }
        winningProposalId = id; 
        emit VotesTallied();
        status = WorkflowStatus.VotesTallied;
    }


    //return le statut en cours du vote
    function getEnum() public view returns(WorkflowStatus){}
 }

   
    
    
    
