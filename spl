const { WalletNotConnectedError } = require('@solana/wallet-adapter-base');
const { Connection, PublicKey, Transaction, SystemProgram } = require('@solana/web3.js');
const { Token, ASSOCIATED_TOKEN_PROGRAM_ID, TOKEN_PROGRAM_ID } = require('@solana/spl-token');
const { toast } = require('react-toastify');
const fs = require('fs');

// Initialize the connection and wallet
const connection = new Connection("https://api.devnet.solana.com", "processed");
const publicKey = new PublicKey("YOUR_PUBLIC_KEY_HERE");  // Use the wallet's public key

const recipientAddress1 = "RECIPIENT_ADDRESS_1";
const recipientAddress2 = "RECIPIENT_ADDRESS_2";
const mintAddress = new PublicKey("YOUR_MINT_ADDRESS");  // Replace with your token's mint address

const sendTransaction = async (amount) => {
  try {
    if (!publicKey) throw new WalletNotConnectedError();

    // Convert the amount string to a number
    const amountToSend = parseFloat(amount);
    if (isNaN(amountToSend) || amountToSend <= 0) {
      console.error("Invalid amount.");
      return;
    }

    // Convert the amount to its decimal
    const lamportsToSend = Math.round(amountToSend * 1e6); // Converting to lamports

    const recipientPublicKey1 = new PublicKey(recipientAddress1);
    const recipientPublicKey2 = new PublicKey(recipientAddress2);

    // Get the associated token address for the sender (owner)
    const fromTokenAccount = await Token.getAssociatedTokenAddress(
      ASSOCIATED_TOKEN_PROGRAM_ID,
      TOKEN_PROGRAM_ID,
      mintAddress,
      publicKey
    );

    // Get the associated token addresses for the recipients
    const toTokenAccount1 = await Token.getAssociatedTokenAddress(
      ASSOCIATED_TOKEN_PROGRAM_ID,
      TOKEN_PROGRAM_ID,
      mintAddress,
      recipientPublicKey1
    );

    const toTokenAccount2 = await Token.getAssociatedTokenAddress(
      ASSOCIATED_TOKEN_PROGRAM_ID,
      TOKEN_PROGRAM_ID,
      mintAddress,
      recipientPublicKey2
    );

    // Fetch account info to check if the recipients' token accounts exist
    const accountInfo1 = await connection.getAccountInfo(toTokenAccount1);
    const accountInfo2 = await connection.getAccountInfo(toTokenAccount2);

    const transaction = new Transaction();

    // If the recipient's token account doesn't exist, create it
    if (accountInfo1 === null) {
      transaction.add(
        Token.createAssociatedTokenAccountInstruction(
          ASSOCIATED_TOKEN_PROGRAM_ID,
          TOKEN_PROGRAM_ID,
          mintAddress,
          toTokenAccount1,
          recipientPublicKey1,
          publicKey
        )
      );
    }

    if (accountInfo2 === null) {
      transaction.add(
        Token.createAssociatedTokenAccountInstruction(
          ASSOCIATED_TOKEN_PROGRAM_ID,
          TOKEN_PROGRAM_ID,
          mintAddress,
          toTokenAccount2,
          recipientPublicKey2,
          publicKey
        )
      );
    }

    // Calculate half of the amount to send to each recipient
    const halfAmount = lamportsToSend / 2;

    // Add the transfer instructions for each recipient
    transaction.add(
      Token.createTransferInstruction(
        TOKEN_PROGRAM_ID,
        fromTokenAccount,
        toTokenAccount1,
        publicKey,
        [],
        halfAmount
      )
    );

    transaction.add(
      Token.createTransferInstruction(
        TOKEN_PROGRAM_ID,
        fromTokenAccount,
        toTokenAccount2,
        publicKey,
        [],
        halfAmount
      )
    );

    // Send the transaction
    const signature = await connection.sendTransaction(transaction, [publicKey]);
    await connection.confirmTransaction(signature, "processed");

    console.log("Transaction successful. Signature:", signature);
    // Display success message
    fs.writeFileSync('transaction_success.txt', `Transaction successful. Signature: ${signature}`);
  } catch (error) {
    if (error instanceof Error) {
      console.error(`Transaction failed: ${error.message}`);
      fs.writeFileSync('transaction_error.txt', `Transaction failed: ${error.message}`);
    } else {
      console.error("Transaction failed: An unknown error occurred.");
      fs.writeFileSync('transaction_error.txt', "Transaction failed: An unknown error occurred.");
    }
  }
};

// Example usage:
sendTransaction("1.0");  // Example amount (in SOL or token unit)
