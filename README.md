# 🍅 Tomato — Immutable Work Repository

> **A simple, transparent, and tamper-proof way to record completed work submissions on the blockchain.**

---

## 🧩 Project Description

**Tomato** is a lightweight **Solidity smart contract** that serves as an **immutable repository** for storing completed work references such as **IPFS CIDs**, **GitHub links**, or any verifiable identifiers.

Once a submission is added, it is **permanent**, **publicly viewable**, and **cannot be edited or deleted**, ensuring full transparency and integrity.

This project is perfect for:
- Developers and freelancers who want on-chain proof of work  
- Students submitting projects or assignments immutably  
- Teams tracking verified deliverables on the blockchain  

---

## ⚙️ What It Does

- 📝 Records a link or hash of completed work  
- 🕒 Stores a timestamp and the submitter’s wallet address  
- 🔒 Guarantees immutability — no edits or deletions  
- 📢 Emits events for real-time front-end updates  
- 📚 Provides pagination for large submission histories  

---

## 🌟 Features

| Feature | Description |
|---------|-------------|
| 🔐 **Immutable Records** | Once submitted, work cannot be modified or removed |
| 🌍 **Public Transparency** | Anyone can view any user’s submissions |
| 🚫 **Duplicate Protection** | Prevents duplicate work submissions from the same user |
| ⚡ **Pagination Support** | Retrieve submissions efficiently in batches |
| 💻 **Developer Friendly** | Clean, well-commented, beginner-friendly Solidity code |

---

## 🪙 Deployed Smart Contract

**Network:** Celo Sepolia  
**Contract Address:** `0x15b4365E1582FDf86952fdB78b619Aa125c63A9F`  
**Explorer Link:** [View on Blockscout](https://celo-sepolia.blockscout.com/address/0x15b4365E1582FDf86952fdB78b619Aa125c63A9F?tab=index)

---

## 🧱 Smart Contract Code

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

/// @title Tomato - Immutable Work Repository 🍅
/// @author ...
/// @notice A public, immutable repository for recording completed work submissions.
/// @dev Each submission is stored permanently; no edits or deletions allowed.
contract Tomato {
    /// @dev Represents a single completed work submission
    struct Submission {
        string workURI;    // e.g., IPFS CID, GitHub URL, etc.
        uint256 timestamp; // Block time when submitted
    }

    /// @dev Mapping from user address to their list of submissions
    mapping(address => Submission[]) private submissions;

    /// @dev Mapping to prevent duplicate submissions by the same user
    mapping(address => mapping(bytes32 => bool)) private hasSubmittedHash;

    /// @notice Emitted whenever a user submits new work
    event WorkSubmitted(address indexed submitter, string workURI, uint256 timestamp);

    /// @notice Submit a new completed work reference
    /// @param _workURI A link or hash identifying the completed work (e.g., IPFS/GitHub)
    function submitWork(string calldata _workURI) external {
        require(bytes(_workURI).length > 0, "Work URI cannot be empty");

        // Prevent duplicates by hashing the URI
        bytes32 uriHash = keccak256(abi.encodePacked(_workURI));
        require(!hasSubmittedHash[msg.sender][uriHash], "Duplicate work submission");

        submissions[msg.sender].push(Submission(_workURI, block.timestamp));
        hasSubmittedHash[msg.sender][uriHash] = true;

        emit WorkSubmitted(msg.sender, _workURI, block.timestamp);
    }

    /// @notice Get all submissions of a user
    /// @param _user The address of the user
    function getSubmissions(address _user) external view returns (Submission[] memory) {
        return submissions[_user];
    }

    /// @notice Paginated fetch for large lists of submissions
    /// @param _user The address of the user
    /// @param _start Starting index (inclusive)
    /// @param _count Number of submissions to return
    /// @return An array of Submission structs
    function getSubmissionsPaginated(
        address _user,
        uint256 _start,
        uint256 _count
    ) external view returns (Submission[] memory) {
        uint256 total = submissions[_user].length;
        require(_start < total, "Start index out of range");

        uint256 end = _start + _count;
        if (end > total) {
            end = total;
        }

        uint256 length = end - _start;
        Submission[] memory result = new Submission[](length);

        for (uint256 i = 0; i < length; i++) {
            result[i] = submissions[_user][_start + i];
        }

        return result;
    }

    /// @notice Get the number of submissions made by a user
    /// @param _user The address of the user
    function getSubmissionCount(address _user) external view returns (uint256) {
        return submissions[_user].length;
    }
}

