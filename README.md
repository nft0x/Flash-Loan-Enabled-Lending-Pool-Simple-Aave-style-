# Flash-Loan-Enabled-Lending-Pool-Simple-Aave-style-
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract FlashLendingPool is ReentrancyGuard, Ownable {
    IERC20 public lendingToken;
    uint256 public flashFee = 9; // 0.09%

    event FlashLoan(address indexed receiver, uint256 amount, uint256 fee);

    constructor(address _token, address initialOwner) Ownable(initialOwner) {
        lendingToken = IERC20(_token);
    }

    function flashLoan(address receiver, uint256 amount, bytes calldata data) external nonReentrant {
        uint256 balanceBefore = lendingToken.balanceOf(address(this));
        require(balanceBefore >= amount, "Insufficient liquidity");

        uint256 fee = (amount * flashFee) / 10000;
        lendingToken.transfer(receiver, amount);

        // Execute receiver callback
        (bool success,) = receiver.call(data);
        require(success, "Flash loan execution failed");

        uint256 balanceAfter = lendingToken.balanceOf(address(this));
        require(balanceAfter >= balanceBefore + fee, "Flash loan not repaid");

        emit FlashLoan(receiver, amount, fee);
    }

    function deposit(uint256 amount) external {
        lendingToken.transferFrom(msg.sender, address(this), amount);
    }
}
