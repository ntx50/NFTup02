How to write and deploy a NFTs smart contract in NEAR (Rust-lang)


Introduction
In this tutorial, we are going to write and test a NFTs smart contract using NEAR protocol with Rust-lang. Then we will deploy it to NEAR Testnet.
Why Rust? Rust is the preferred programming language for writing smart contracts on NEAR. Rust offers many features like memory safety, small runtime, etc. Rust allows us to write a smart contract that doesn’t have memory bugs and consumes less storage on the blockchain.

Requirements
Requirements installed:
Rust (Installation guid here)
NEAR CLI (Installation guide)
NEAR Testnet account (If you don't have testnet account, create guide HERE)

Setup
To  a new NEAR project with default settings, you just need one command
Using npm's npx:
npx create-near-app --frontend=react --contract=rust new-awesome-project-name

Or, if you prefer yarn:
yarn create near-app --frontend=react --contract=rust new-awesome-project-name


Writing the contract
We can start by removing all the existing code in contract/src/ib.rs and pasting in the following code snippet:
//Rust Software development kit(SDK)library for writing NEAR smart contracts NFT
use near_contract_standards::non_fungible_token::metadata::{
   NFTContractMetadata, NonFungibleTokenMetadataProvider, TokenMetadata, NFT_METADATA_SPEC,
};
use near_contract_standards::non_fungible_token::{Token, TokenId};
use near_contract_standards::non_fungible_token::NonFungibleToken;
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::collections::LazyOption;
use near_sdk::json_types::ValidAccountId;
use near_sdk::{
   env, near_bindgen, AccountId, BorshStorageKey, PanicOnDefault, Promise, PromiseOrValue,
};
// 1. Main Struct
// 2. Default Implementation
// 3. Core Logic


At the top of the contract, we need to import a few code modules with the use declaration. We will expand on these portions of the near_sdk below.
Next, we setup the global allocator from the wee_alloc crate using the setup_alloc!() macro. Allocators are the way that programs in Rust obtain memory from the system at runtime. 


Main struct
While writing our smart contract, we will follow a pattern using one structure (struct) and an implementation (impl) associated with it. This is a pattern used in most Rust contracts on NEAR. Add the following snippet below the comment // 1. Main Struct in lib.rs:
// 1. Main Struct
#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
pub struct Contract {
   tokens: NonFungibleToken,
   metadata: LazyOption<NFTContractMetadata>,
}
 
#[derive(BorshSerialize, BorshStorageKey)]
enum StorageKey {
   NonFungibleToken,
   Metadata,
   TokenMetadata,
   Enumeration,
   Approval,
}

We have our main structure Contract which has two fields token and metadata which we have imported from near_sdk::collections and near_contract_standards::non_fungible_token::
To get a better overview of all the available ways storage can be used, check out documentation of collections module.
Check out enum  for enum StorageKey.
#[near_bindgen] and #[derive(BorshDeserialize, BorshSerialize)] are attributes.
A declarative tag that is used to convey information to runtime about the behaviors of various elements like classes, methods, structures, enumerators, assemblies, etc.
By adding the macro #[near_bindgen] we provide our struct KeyValue with generated boilerplate code to make it compatible with the NEAR blockchain. The second macro, #[derive(BorshDeserialize, BorshSerialize)] assists in the serialization and de-serialization of the data for sending it to or retrieiving it from NEAR.
we will use some //Crate near_contract_standards:
//Crate near_contract_standards
//The core methods for a basic non-fungible token.
near_contract_standards::impl_non_fungible_token_core!(Contract, tokens);
//Non-fungible token approval management allows for an escrow system where multiple approvals per token exist.
near_contract_standards::impl_non_fungible_token_approval!(Contract, tokens);
//Non-fungible enumeration adds the extension standard offering several view-only methods to get token supply, tokens per owner, etc.
near_contract_standards::impl_non_fungible_token_enumeration!(Contract, tokens);


Default implementation
Every type in Rust has a Default implementation but here we want provide our own default implementation for Contract struct. Add the following snippet below the comment // 2. Default Implementation in lib.rs:
// 2. Default Implementation
#[near_bindgen]
impl NonFungibleTokenMetadataProvider for Contract {
   fn nft_metadata(&self) -> NFTContractMetadata {
       self.metadata.get().unwrap()
   }

First, we are creating a Default implementation for Contract. After that, we added the default method inside that implementation which returns NFTContractMetadata. Lastly, we are returning Self with metadata.get().unwrap().

Core logic
Now we are going to add methods to Contract struct. These methods are core logic for our smart contract. Add following snippet below the comment // 3. Core Logic:
// 3. Core Logic
#[near_bindgen]
impl Contract {
   #[init]
   pub fn new_default_meta(owner_id: ValidAccountId) -> Self {
       Self::new(
           owner_id,
           NFTContractMetadata {
               spec: NFT_METADATA_SPEC.to_string(),
               name: "NFTup".to_string(),
               symbol: "NFT".to_string(),
               icon: None,
               base_uri: None,
               reference: None,
               reference_hash: None,
           },
       )
   }


When creating methods, we must have an implementation block defined by the impl keyword followed by the name of the struct to be implemented. The pub keyword makes the methods publicly available, meaning they can be called by anyone with access to the protocol and a means of signing the transaction.

The first method new_default_meta cross contract to method new 

 






   pub fn new(owner_id: ValidAccountId, metadata: NFTContractMetadata) -> Self {
       assert!(!env::state_exists(), "Already initialized");
       metadata.assert_valid();
       Self {
           tokens: NonFungibleToken::new(
               StorageKey::NonFungibleToken,
               owner_id,
               Some(StorageKey::TokenMetadata),
               Some(StorageKey::Enumeration),
               Some(StorageKey::Approval),
           ),
           metadata: LazyOption::new(StorageKey::Metadata, Some(&metadata)),
       }
   }
 


To get a better overview of all the cross smartcontract check out documentation 


    // Payable anotation allow tokens to be minting with the method invocation 
    #[payable]
    pub fn nft_mint(
        &mut self,
        token_id: TokenId,
        receiver_id: ValidAccountId,
        token_metadata: TokenMetadata,
    ) -> Token {
        self.tokens.mint(token_id, receiver_id, Some(token_metadata))
    }


After that, we are calling the method nft_mint. This will create a Token NFTs.

We are using &mut self to borrow self mutably. You can learn more about borrowing here.

 
 
Every smart contract in NEAR has its own associated account. When you run yarn dev, your smart contract gets deployed to the live NEAR TestNet with a throwaway account. When you're ready to make it permanent, here's how.
Step 0: Install near-cli (optional)
near-cli is a command line interface (CLI) for interacting with the NEAR blockchain. It was installed to the local node_modules folder when you ran yarn install, but for best ergonomics you may want to install it globally:
yarn install --global near-cli
 
Or, if you'd rather use the locally-installed version, you can prefix all near commands with npx
Ensure that it's installed with near --version (or npx near --version)
Step 1: Create an account for the contract
Each account on NEAR can have at most one contract deployed to it. If you've already created an account such as your-name.testnet, you can deploy your contract to challenge-5.your-name.testnet. Assuming you've already created an account on NEAR Wallet, here's how to create challenge-5.your-name.testnet:
Authorize NEAR CLI, following the commands it gives you:
near login
Create a subaccount (replace YOUR-NAME below with your actual account name):
near create-account challenge-5.YOUR-NAME.testnet --masterAccount YOUR-NAME.testnet
Step 2: set contract name in code
Modify the line in src/config.js that sets the account name of the contract. Set it to the account id you used above.
const CONTRACT_NAME = process.env.CONTRACT_NAME || 'challenge-5.YOUR-NAME.testnet'
 
Step 3: deploy!
One command:
yarn deploy
 
As you can see in package.json, this does two things:
builds & deploys smart contract to NEAR TestNet
builds & deploys frontend code to GitHub using gh-pages. This will only work if the project already has a repository set up on GitHub. Feel free to modify the deploy script in package.json to deploy elsewhere.





