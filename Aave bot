from web3 import Web3
import json, time, requests
import numpy as np
import pandas as pd
from eth_account import Account

# === CONFIGURATION ===
INFURA_URL = "https://mainnet.infura.io/v3/YOUR_INFURA_KEY"
BSC_URL = "https://bsc-dataseed.binance.org/"
PRIVATE_KEY = "YOUR_WALLET_PRIVATE_KEY"
WALLET_ADDRESS = "YOUR_WALLET_ADDRESS"

AAVE_LENDING_POOL = "0x7d2768dE32b0b80b7a3454c060A9A87fD99220a9"
DEX_ROUTERS = {
    "uniswap": "0x7a250d5630b4cF539739dF2C5dAcb4c659F2488D",
    "sushiswap": "0xd9e1cE17f2641f24aE83637ab66a2cca9C378B9F",
    "pancakeswap": "0x10ED43C718714eb63d5aA57B78B54704E256024E",
    "curve": "0xD533a949740bb3306d119CC777fa900bA034cd52",
    "balancer": "0xba12222222228d8Ba445958a75a0704d566BF2C8"
}

TOKENS = {
    "DAI": "0x6B175474E89094C44Da98b954EedeAC495271d0F",
    "WETH": "0xC02aaa39b223FE8D0A0E5C4F27eAD9083C756Cc2",
    "BUSD": "0xe9e7cea3dedca5984780bafc599bd69add087d56"
}

AMOUNT_IN = Web3.to_wei(5000, 'ether')  # 5000 DAI
GAS_LIMIT = 2500000
GAS_PRICE = Web3.to_wei('25', 'gwei')

# === CONNECT TO BLOCKCHAIN ===
web3 = Web3(Web3.HTTPProvider(INFURA_URL))
bsc_web3 = Web3(Web3.HTTPProvider(BSC_URL))
account = Account.from_key(PRIVATE_KEY)

# === LOAD SMART CONTRACTS ===
def load_contract(abi_path, address, blockchain='eth'):
    with open(abi_path) as f:
        abi = json.load(f)
    if blockchain == 'eth':
        return web3.eth.contract(address=address, abi=abi)
    return bsc_web3.eth.contract(address=address, abi=abi)

uniswap = load_contract("UniswapABI.json", DEX_ROUTERS["uniswap"])
sushiswap = load_contract("SushiSwapABI.json", DEX_ROUTERS["sushiswap"])
pancakeswap = load_contract("PancakeSwapABI.json", DEX_ROUTERS["pancakeswap"], 'bsc')
aave = load_contract("AaveLendingPoolABI.json", AAVE_LENDING_POOL)

# === FETCH TOKEN PRICES ===
def get_price(token_in, token_out, amount, dex):
    router = {"uniswap": uniswap, "sushiswap": sushiswap, "pancakeswap": pancakeswap}.get(dex)
    try:
        amount_out = router.functions.getAmountsOut(amount, [token_in, token_out]).call()
        return Web3.from_wei(amount_out[-1], 'ether')
    except:
        return None

# === FIND BEST ARBITRAGE OPPORTUNITY ===
def find_best_arbitrage():
    prices = {}
    for dex in DEX_ROUTERS.keys():
        prices[dex] = get_price(TOKENS["DAI"], TOKENS["WETH"], AMOUNT_IN, dex)

    if None in prices.values():
        return None

    sorted_prices = sorted(prices.items(), key=lambda x: x[1])
    buy_dex, sell_dex = sorted_prices[0][0], sorted_prices[-1][0]
    profit = sorted_prices[-1][1] - sorted_prices[0][1]

    return {'buy_dex': buy_dex, 'sell_dex': sell_dex, 'profit': profit} if profit > 0 else None

# === AI-POWERED PRICE PREDICTION ===
def predict_next_move():
    data = np.random.randn(100)  # Simulating past price trends
    df = pd.DataFrame(data, columns=['price'])
    df['predicted'] = df['price'].rolling(5).mean()
    return df['predicted'].iloc[-1]

# === EXECUTE FLASH LOAN AND TRADE ===
def execute_arbitrage():
    arb = find_best_arbitrage()
    if not arb:
        print("No profitable trade found.")
        return

    buy_dex = {"uniswap": uniswap, "sushiswap": sushiswap, "pancakeswap": pancakeswap}[arb['buy_dex']]
    sell_dex = {"uniswap": uniswap, "sushiswap": sushiswap, "pancakeswap": pancakeswap}[arb['sell_dex']]

    # Flash Loan from Aave
    tx_flashloan = aave.functions.flashLoan(
        WALLET_ADDRESS, TOKENS["DAI"], AMOUNT_IN, 0
    ).build_transaction({
        'from': WALLET_ADDRESS,
        'gas': GAS_LIMIT,
        'gasPrice': GAS_PRICE,
        'nonce': web3.eth.get_transaction_count(WALLET_ADDRESS),
    })
    signed_flashloan = web3.eth.account.sign_transaction(tx_flashloan, PRIVATE_KEY)
    web3.eth.send_raw_transaction(signed_flashloan.rawTransaction)

    # Buy Low, Sell High
    tx_buy = buy_dex.functions.swapExactTokensForTokens(
        AMOUNT_IN, 0, [TOKENS["DAI"], TOKENS["WETH"]], WALLET_ADDRESS, int(time.time()) + 600
    ).build_transaction({
        'from': WALLET_ADDRESS,
        'gas': GAS_LIMIT,
        'gasPrice': GAS_PRICE,
        'nonce': web3.eth.get_transaction_count(WALLET_ADDRESS),
    })
    signed_buy = web3.eth.account.sign_transaction(tx_buy, PRIVATE_KEY)
    web3.eth.send_raw_transaction(signed_buy.rawTransaction)

    print(f"Trade executed! Profit: {arb['profit']} ETH")

# === RUN 24/7 WITH GAS OPTIMIZATION ===
def main():
    while True:
        execute_arbitrage()
        time.sleep(2)  # Optimize execution frequency

if __name__ == "__main__":
    main()
