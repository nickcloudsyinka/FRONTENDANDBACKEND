# Children Voting DApp

This project is a Children decentralized application (DApp) built with React and Web3.js. It interacts with an Ethereum smart contract to allow users to add proposals, vote on them, and even vote through another contract.

## Project Structure

- **React**: Frontend framework used to build the user interface.
- **Web3.js**: JavaScript library to interact with the Ethereum blockchain.
- **MetaMask**: Ethereum wallet required to interact with the DApp.

## Prerequisites

Before running this project, ensure you have the following installed:

- **Node.js**: [Download Node.js](https://nodejs.org/)
- **MetaMask**: [Install MetaMask](https://metamask.io/)
- **Ethereum Smart Contract**: A deployed Ethereum smart contract with the necessary ABI.

## Setup

1. **Clone the repository**:

    ```bash
    git clone https://github.com/your-repo/simple-voting-dapp.git
    cd simple-voting-dapp
    ```

2. **Install dependencies**:

    ```bash
    npm install
    ```

3. **Configure the contract ABI and address**:

    - Replace the placeholder `contractABI` in the `App.js` file with your contract's ABI.
    - Replace `contractAddress` with your deployed contract's address.

    ```javascript
    const contractABI = [ /* Your contract ABI here */ ];
    const contractAddress = '0xYourContractAddressHere';
    ```

4. **Run the application**:

    ```bash
    npm start
    ```

5. **Access the DApp**:

    - Open your browser and go to `http://localhost:3000/`.
    - Ensure MetaMask is installed and connected to the correct potentially test Ethereum network.

## Usage

### Adding a Proposal

1. In the "Add Children" section, enter a proposal name in the input field.
2. Click the "Add Children" button to submit the proposal.
3. The proposal will be added to the list after the transaction is confirmed.

### Viewing Proposals

1. The "Children" section displays all current Children.
2. Each proposal shows its ID, name, and the number of votes it has received.

### Voting Directly

1. In the "Vote" section, enter the ID of the Children you want to vote for.
2. Click the "Vote Directly" button to cast your vote.
3. The vote count for the Children will increase after the transaction is confirmed.

### Voting via Another Contract

1. In the "Vote with Contract" section, enter the ID of the Children you want to vote for.
2. Enter the address of the contract you wish to vote through.
3. Click the "Vote via Contract" button to cast your vote.
4. The vote count for the proposal will increase after the transaction is confirmed.

## Code Overview

### `App.js`

- **State Variables**:
  - `web3`: Instance of Web3.
  - `account`: The currently connected Ethereum account.
  - `contract`: The smart contract instance.
  - `proposals`: An array of proposals fetched from the contract.
  - `proposalName`: Stores the name of the new proposal to be added.
  - `proposalId`: Stores the ID of the proposal to vote on.
  - `votingContract`: Stores the address of another contract to vote through.

- **Functions**:
  - `loadWeb3()`: Initializes Web3, connects to MetaMask, and loads contract data.
  - `handleAddProposal()`: Adds a new proposal to the contract.
  - `handleVote()`: Votes directly on a proposal.
  - `handleVoteWithContract()`: Votes on a proposal through another contract.

## Notes

- Ensure that your MetaMask is connected to the same Ethereum network where your contract is deployed.
- The DApp will not function without a valid contract ABI and address.

## License

This project is licensed under the MIT License.

```js
import React, { useState, useEffect } from 'react';
import Web3 from 'web3';
import SimpleVotingABI from './SimpleVotingABI.json';
import './App.css';

const contractABI = SimpleVotingABI;
const contractAddress = "0x048DC24fD4841B471aFfD490287F088BEe26219e";

function SimpleVoting() {
    const [web3, setWeb3] = useState(null);
    const [account, setAccount] = useState('');
    const [contract, setContract] = useState(null);
    const [proposals, setProposals] = useState([]);
    const [proposalName, setProposalName] = useState('');
    const [proposalId, setProposalId] = useState('');
    const [votingContract, setVotingContract] = useState('');
    const [successMessage, setSuccessMessage] = useState('');

    useEffect(() => {
        async function loadWeb3() {
            if (window.ethereum) {
                const web3Instance = new Web3(window.ethereum);
                setWeb3(web3Instance);
                await window.ethereum.enable();

                const accounts = await web3Instance.eth.getAccounts();
                setAccount(accounts[0]);

                const contractInstance = new web3Instance.eth.Contract(contractABI, contractAddress);
                setContract(contractInstance);

                try {
                    const proposalCount = await contractInstance.methods.proposalCount().call();
                    let proposalsArray = [];
                    for (let i = 1; i <= proposalCount; i++) {
                        const proposal = await contractInstance.methods.proposals(i).call();
                        proposalsArray.push(proposal);
                    }
                    setProposals(proposalsArray);
                } catch (error) {
                    console.error("Error fetching Children:", error);
                    alert(`Error fetching Children: ${error.message || error}`);
                }
            } else {
                alert('Please install MetaMask to use this dApp!');
            }
        }
        loadWeb3();
    }, []);

    const handleAddProposal = async () => {
        if (proposalName && contract) {
            try {
                await contract.methods.addProposal(proposalName).send({ from: account });
                setSuccessMessage('Children added successfully!');
                setTimeout(() => setSuccessMessage(''), 5000); // Clear message after 5 seconds
                loadProposals(); // Refresh proposals
            } catch (error) {
                console.error("Error adding Children:", error);
                alert(`Error adding Children: ${error.message || error}`);
            }
        } else {
            alert("Children name cannot be empty!");
        }
    };

    const handleVote = async () => {
        if (proposalId && contract) {
            try {
                await contract.methods.vote(proposalId).send({ from: account });
                setSuccessMessage('Children Vote cast successfully!');
                setTimeout(() => setSuccessMessage(''), 5000); // Clear message after 5 seconds
                loadProposals(); // Refresh proposals
            } catch (error) {
                console.error("Error voting:", error);
                alert(`Error voting: ${error.message || error}`);
            }
        } else {
            alert("Children ID cannot be empty!");
        }
    };

    const handleVoteWithContract = async () => {
        if (proposalId && votingContract && contract) {
            try {
                await contract.methods.voteWithContract(proposalId, votingContract).send({ from: account });
                setSuccessMessage('Vote cast via contract successfully!');
                setTimeout(() => setSuccessMessage(''), 5000); // Clear message after 5 seconds
                loadProposals(); // Refresh proposals
            } catch (error) {
                console.error("Error voting with contract:", error);
                alert(`Error voting with contract: ${error.message || error}`);
            }
        } else {
            alert("Children ID and Voting Contract Address cannot be empty!");
        }
    };

    const handleDeleteVote = async () => {
        if (proposalId && contract) {
            try {
                await contract.methods.deleteVote(proposalId).send({ from: account });
                setSuccessMessage('Vote deleted successfully!');
                setTimeout(() => setSuccessMessage(''), 5000); // Clear message after 5 seconds
                loadProposals(); // Refresh proposals
            } catch (error) {
                console.error("Error deleting vote:", error);
                alert(`Error deleting vote: ${error.message || error}`);
            }
        } else {
            alert("Children ID cannot be empty!");
        }
    };

    const handleAddMoreVotes = async () => {
        if (proposalId && contract) {
            const amount = prompt('Enter the amount of votes to add:');
            if (amount) {
                try {
                    await contract.methods.addMoreVotes(proposalId, amount).send({ from: account });
                    setSuccessMessage(`${amount} votes added successfully!`);
                    setTimeout(() => setSuccessMessage(''), 5000); // Clear message after 5 seconds
                    loadProposals(); // Refresh proposals
                } catch (error) {
                    console.error("Error adding more votes:", error);
                    alert(`Error adding more votes: ${error.message || error}`);
                }
            } else {
                alert('Amount cannot be empty!');
            }
        } else {
            alert("Children ID cannot be empty!");
        }
    };

    const handleGetTotalVotes = async () => {
        if (contract) {
            try {
                const totalVotes = await contract.methods.getTotalVotes().call();
                alert(`Total Votes: ${totalVotes}`);
            } catch (error) {
                console.error("Error getting total votes:", error);
                alert(`Error getting total votes: ${error.message || error}`);
            }
        }
    };

    const handleRemoveVote = async () => {
        if (proposalId && contract) {
            try {
                await contract.methods.removeVote(proposalId).send({ from: account });
                setSuccessMessage('Vote removed successfully!');
                setTimeout(() => setSuccessMessage(''), 5000); // Clear message after 5 seconds
                loadProposals(); // Refresh proposals
            } catch (error) {
                console.error("Error removing vote:", error);
                alert(`Error removing vote: ${error.message || error}`);
            }
        } else {
            alert("Children ID cannot be empty!");
        }
    };

    const loadProposals = async () => {
        try {
            const proposalCount = await contract.methods.proposalCount().call();
            let proposalsArray = [];
            for (let i = 1; i <= proposalCount; i++) {
                const proposal = await contract.methods.proposals(i).call();
                proposalsArray.push(proposal);
            }
            setProposals(proposalsArray);
        } catch (error) {
            console.error("Error fetching Children:", error);
            alert(`Error fetching Children: ${error.message || error}`);
        }
    };

    return (
        <div className='bg'>
            <h1>Children Voting DApp</h1>
            <p>Connected Account: {account}</p>

            <h2>Add Children Name</h2>
            <input 
                type="text" 
                placeholder="Children Name" 
                value={proposalName} 
                onChange={(e) => setProposalName(e.target.value)} 
            />
            <button onClick={handleAddProposal}>Add Children Name</button>

            <h2>Children</h2>
            <ul>
                {proposals.map((proposal) => (
                    <li key={proposal.id}>
                        {proposal.id}. {proposal.name} - {proposal.voteCount} votes
                    </li>
                ))}
            </ul>

            <h2>Pick A Vote</h2>
            <input 
                type="number" 
                placeholder="Children ID" 
                value={proposalId} 
                onChange={(e) => setProposalId(e.target.value)} 
            />
            <button onClick={handleVote}>Children Vote Directly</button>

            <h2>Add Vote Children</h2>
            <input 
                type="text" 
                placeholder="Voting Contract Address" 
                value={votingContract} 
                onChange={(e) => setVotingContract(e.target.value)} 
            />
            <button onClick={handleVoteWithContract}>Vote via Contract Of The Children</button>

            <h2>Delete Vote Children</h2>
            <button onClick={handleDeleteVote}>Delete Vote Of The Children Name</button>

            <h2>Add More Votes To one off The Children</h2>
            <button onClick={handleAddMoreVotes}>Add More Votes To One Off The Children</button>

            <h2>Total Votes For A Children</h2>
            <button onClick={handleGetTotalVotes}>Get Total Votes Off The Children</button>

            <h2>Remove Vote of The Children</h2>
            <button onClick={handleRemoveVote}>Remove Vote</button>

            {successMessage && <p>{successMessage}</p>}
        </div>
    );
}

export default SimpleVoting;
