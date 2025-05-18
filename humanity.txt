from web3 import Web3
from colorama import init, Fore, Style
import sys
import time

# Initialize colorama
init(autoreset=True)

# Function to display the header
def display_header():
    print(Fore.CYAN + Style.BRIGHT + "===============================")
    print(Fore.YELLOW + Style.BRIGHT + "Auto Daily Claim $RWT Humanity Protocol")
    print(Fore.CYAN + Style.BRIGHT + "Bot created by: " + Fore.GREEN + "https://t.me/airdropwithmeh")
    print(Fore.CYAN + Style.BRIGHT + "===============================\n")

# Function to connect to the blockchain network
def connect_to_network():
    rpc_url = 'https://rpc.testnet.humanity.org'
    web3 = Web3(Web3.HTTPProvider(rpc_url))
    if web3.is_connected():
        print(Fore.GREEN + "Connected to Humanity Protocol")
        return web3
    else:
        print(Fore.RED + "Connection failed.")
        sys.exit(1)

# Smart contract address and ABI
contract_address = '0xa18f6FCB2Fd4884436d10610E69DB7BFa1bFe8C7'
contract_abi = [{"inputs":[],"name":"AccessControlBadConfirmation","type":"error"},{"inputs":[{"internalType":"address","name":"account","type":"address"},{"internalType":"bytes32","name":"neededRole","type":"bytes32"}],"name":"AccessControlUnauthorizedAccount","type":"error"},{"inputs":[],"name":"InvalidInitialization","type":"error"},{"inputs":[],"name":"NotInitializing","type":"error"},{"anonymous":False,"inputs":[{"indexed":False,"internalType":"uint64","name":"version","type":"uint64"}],"name":"Initialized","type":"event"},{"anonymous":False,"inputs":[{"indexed":True,"internalType":"address","name":"from","type":"address"},{"indexed":True,"internalType":"address","name":"to","type":"address"},{"indexed":False,"internalType":"uint256","name":"amount","type":"uint256"},{"indexed":False,"internalType":"bool","name":"bufferSafe","type":"bool"}],"name":"ReferralRewardBuffered","type":"event"},{"anonymous":False,"inputs":[{"indexed":True,"internalType":"address","name":"user","type":"address"},{"indexed":True,"internalType":"enum IRewards.RewardType","name":"rewardType","type":"uint8"},{"indexed":False,"internalType":"uint256","name":"amount","type":"uint256"}],"name":"RewardClaimed","type":"event"},{"anonymous":False,"inputs":[{"indexed":True,"internalType":"bytes32","name":"role","type":"bytes32"},{"indexed":True,"internalType":"bytes32","name":"previousAdminRole","type":"bytes32"},{"indexed":True,"internalType":"bytes32","name":"newAdminRole","type":"bytes32"}],"name":"RoleAdminChanged","type":"event"},{"anonymous":False,"inputs":[{"indexed":True,"internalType":"bytes32","name":"role","type":"bytes32"},{"indexed":True,"internalType":"address","name":"account","type":"address"},{"indexed":True,"internalType":"address","name":"sender","type":"address"}],"name":"RoleGranted","type":"event"},{"anonymous":False,"inputs":[{"indexed":True,"internalType":"bytes32","name":"role","type":"bytes32"},{"indexed":True,"internalType":"address","name":"account","type":"address"},{"indexed":True,"internalType":"address","name":"sender","type":"address"}],"name":"RoleRevoked","type":"event"},{"inputs":[],"name":"DEFAULT_ADMIN_ROLE","outputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"claimBuffer","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"claimReward","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"currentEpoch","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"cycleStartTimestamp","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"address","name":"user","type":"address"}],"name":"userGenesisClaimStatus","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"view","type":"function"}]  # (Keep the ABI as it is)

# Load the contract
def load_contract(web3):
    return web3.eth.contract(address=Web3.to_checksum_address(contract_address), abi=contract_abi)

# Function to load private keys from a text file
def load_private_keys(file_path):
    with open(file_path, 'r') as file:
        return [line.strip() for line in file if line.strip()]

# Function to claim rewards
def claim_rewards(contract, web3, private_key):
    try:
        account = web3.eth.account.from_key(private_key)
        sender_address = account.address
        reward_claimed = contract.functions.userGenesisClaimStatus(sender_address).call()
        
        if reward_claimed:
            print(Fore.YELLOW + f"Reward already claimed for address: {sender_address}. Skipping.")
            return

        gas_amount = contract.functions.claimReward().estimate_gas({
            'chainId': web3.eth.chain_id,
            'from': sender_address,
            'gasPrice': web3.eth.gas_price,
            'nonce': web3.eth.get_transaction_count(sender_address)
        })

        transaction = contract.functions.claimReward().build_transaction({
            'chainId': web3.eth.chain_id,
            'from': sender_address,
            'gas': gas_amount,
            'gasPrice': web3.eth.gas_price,
            'nonce': web3.eth.get_transaction_count(sender_address)
        })

        signed_txn = web3.eth.account.sign_transaction(transaction, private_key=private_key)
        tx_hash = web3.eth.send_raw_transaction(signed_txn.rawTransaction)
        tx_receipt = web3.eth.wait_for_transaction_receipt(tx_hash)

        print(Fore.GREEN + f"Transaction successful for {sender_address} with hash: {web3.to_hex(tx_hash)}")

    except Exception as e:
        error_message = str(e)
        if "Rewards: user not registered" in error_message:
            print(Fore.RED + f"Error: User {sender_address} is not registered.")
        else:
            print(Fore.RED + f"Error claiming reward for {sender_address}: {error_message}")

# Main execution loop
if __name__ == "__main__":
    while True:
        display_header()
        web3 = connect_to_network()
        contract = load_contract(web3)
        private_keys = load_private_keys('private_keys.txt')
        
        for private_key in private_keys:
            claim_rewards(contract, web3, private_key)

        print(Fore.CYAN + "Waiting for 10 minutes before the next run...")
        time.sleep(600)  # Wait for 10 minutes
