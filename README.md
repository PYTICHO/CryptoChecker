# CryptoChecker
import aiohttp
import asyncio

async def fetch_json(session, url, params=None):
    async with session.get(url, params=params) as response:
        return await response.json()

async def get_prices_htx(session):
    url = "https://api.huobi.pro/market/tickers"
    data = await fetch_json(session, url)
    
    prices = {}
    if 'data' in data:
        for item in data['data']:
            symbol = item['symbol'].upper()
            prices[symbol] = item['close']
    return prices

async def get_prices_bybit(session):
    url = "https://api.bybit.com/v5/market/tickers"
    data = await fetch_json(session, url, params={"category": "spot"})
    
    prices = {}
    if "result" in data and "list" in data["result"]:
        for item in data["result"]["list"]:
            symbol = item['symbol'].upper()
            prices[symbol] = float(item['lastPrice'])
    return prices

async def get_prices_binance(session):
    url = "https://api.binance.com/api/v3/ticker/price"
    data = await fetch_json(session, url)
    
    prices = {}
    for item in data:
        symbol = item['symbol'].upper()
        prices[symbol] = float(item['price'])
    return prices

async def get_prices_okx(session):
    url = "https://www.okx.com/api/v5/market/tickers?instType=SPOT"
    data = await fetch_json(session, url)

    prices = {}
    if "data" in data:
        for item in data['data']:
            symbol = item['instId'].replace("-", "").upper()
            prices[symbol] = float(item['last'])
    return prices

async def get_prices_mexc(session):
    url = "https://www.mexc.com/open/api/v2/market/ticker"
    data = await fetch_json(session, url)

    prices = {}
    if "data" in data:
        for item in data['data']:
            symbol = item['symbol'].upper()
            prices[symbol] = float(item['last'])
    return prices

async def get_prices_gate(session):
    url = "https://api.gateio.ws/api/v4/spot/tickers"
    data = await fetch_json(session, url)

    prices = {}
    for item in data:
        symbol = item['currency_pair'].replace("_", "").upper()
        prices[symbol] = float(item['last'])
    return prices

async def get_prices_bitget(session):
    url = "https://api.bitget.com/api/spot/v1/market/tickers"
    data = await fetch_json(session, url)

    prices = {}
    if "data" in data:
        for item in data['data']:
            symbol = item['symbol'].replace("_", "").upper()
            prices[symbol] = float(item['close'])
    return prices

async def get_withdrawal_deposit_status_htx(session, symbol):
    url = "https://api.huobi.pro/v2/reference/currencies"
    data = await fetch_json(session, url)
    
    if "data" in data:
        for item in data["data"]:
            if item["currency"] == symbol.lower():
                deposit_status = item["deposit-status"]
                withdrawal_status = item["withdraw-status"]
                if deposit_status == "allowed" and withdrawal_status == "allowed":
                    networks = [chain["chain"] for chain in item["chains"]]
                    return {
                        "symbol": symbol,
                        "deposit_enabled": True,
                        "withdrawal_enabled": True,
                        "networks": networks
                    }
                return {
                    "symbol": symbol,
                    "deposit_enabled": deposit_status == "allowed",
                    "withdrawal_enabled": withdrawal_status == "allowed",
                    "networks": []
                }
    return None

async def get_withdrawal_deposit_status_bybit(session, symbol):
    url = "https://api.bybit.com/v5/asset/coin/query"
    data = await fetch_json(session, url)

    if "result" in data and "list" in data["result"]:
        for item in data["result"]["list"]:
            if item["coin"] == symbol.upper():
                deposit_status = item["deposit"]["status"]
                withdrawal_status = item["withdraw"]["status"]
                if deposit_status == 1 and withdrawal_status == 1:
                    networks = [network["chain"] for network in item["chains"]]
                    return {
                        "symbol": symbol,
                        "deposit_enabled": True,
                        "withdrawal_enabled": True,
                        "networks": networks
                    }
                return {
                    "symbol": symbol,
                    "deposit_enabled": deposit_status == 1,
                    "withdrawal_enabled": withdrawal_status == 1,
                    "networks": []
                }
    return None

async def get_withdrawal_deposit_status_binance(session, symbol):
    url = f"https://api.binance.com/sapi/v1/capital/config/getall?coin={symbol.lower()}"
    data = await fetch_json(session, url)

    if isinstance(data, list) and len(data) > 0:
        item = data[0]
        deposit_status = item["depositAllEnable"]
        withdrawal_status = item["withdrawAllEnable"]
        networks = item.get("networkList", [])
        available_networks = [net["network"] for net in networks if net["withdrawEnable"]]
        
        return {
            "symbol": symbol,
            "deposit_enabled": deposit_status,
            "withdrawal_enabled": withdrawal_status,
            "networks": available_networks
        }
    return None

async def get_withdrawal_deposit_status_okx(session, symbol):
    url = f"https://www.okx.com/api/v5/asset/deposit-withdrawal?currency={symbol.lower()}"
    data = await fetch_json(session, url)

    if "data" in data and len(data["data"]) > 0:
        item = data["data"][0]
        deposit_status = item["can_deposit"]
        withdrawal_status = item["can_withdraw"]
        networks = item.get("chains", [])
        return {
            "symbol": symbol,
            "deposit_enabled": deposit_status,
            "withdrawal_enabled": withdrawal_status,
            "networks": networks
        }
    return None

async def get_withdrawal_deposit_status_mexc(session, symbol):
    url = "https://www.mexc.com/open/api/v2/asset/deposit-withdraw"
    data = await fetch_json(session, url)

    if "data" in data:
        for item in data["data"]:
            if item["coin"] == symbol.lower():
                deposit_status = item["isDepositEnabled"]
                withdrawal_status = item["isWithdrawEnabled"]
                networks = item.get("chains", [])
                return {
                    "symbol": symbol,
                    "deposit_enabled": deposit_status,
                    "withdrawal_enabled": withdrawal_status,
                    "networks": networks
                }
    return None

async def get_withdrawal_deposit_status_gate(session, symbol):
    url = f"https://api.gateio.ws/api/v4/asset/deposit-withdrawal?currency={symbol.lower()}"
    data = await fetch_json(session, url)

    if "data" in data and len(data["data"]) > 0:
        item = data["data"][0]
        deposit_status = item["can_deposit"]
        withdrawal_status = item["can_withdraw"]
        networks = item.get("chains", [])
        return {
            "symbol": symbol,
            "deposit_enabled": deposit_status,
            "withdrawal_enabled": withdrawal_status,
            "networks": networks
        }
    return None

async def get_withdrawal_deposit_status_bitget(session, symbol):
    url = f"https://api.bitget.com/api/spot/v1/asset/withdraw?symbol={symbol.lower()}"
    data = await fetch_json(session, url)

    if "data" in data:
        deposit_status = data["data"].get("isDepositEnabled", False)
        withdrawal_status = data["data"].get("isWithdrawEnabled", False)
        networks = data["data"].get("chains", [])
        return {
            "symbol": symbol,
            "deposit_enabled": deposit_status,
            "withdrawal_enabled": withdrawal_status,
            "networks": networks
        }
    return None

async def aggregate_prices_with_statuses():
    async with aiohttp.ClientSession() as session:
        tasks = [
            get_prices_htx(session),
            get_prices_bybit(session),
            get_prices_binance(session),
            get_prices_okx(session),
            get_prices_mexc(session),
            get_prices_gate(session),
            get_prices_bitget(session)
        ]
        results = await asyncio.gather(*tasks)
        
        prices_htx, prices_bybit, prices_binance, prices_okx, prices_mexc, prices_gate, prices_bitget = results
        
        aggregated_prices = {}
        
        # Получаем все уникальные символы из всех цен
        all_symbols = set()
        all_symbols.update(prices_htx.keys())
        all_symbols.update(prices_bybit.keys())
        all_symbols.update(prices_binance.keys())
        all_symbols.update(prices_okx.keys())
        all_symbols.update(prices_mexc.keys())
        all_symbols.update(prices_gate.keys())
        all_symbols.update(prices_bitget.keys())

        # Агрегируем цены и статусы депозитов/выводов
        for symbol in all_symbols:
            aggregated_prices[symbol] = {}

            if symbol in prices_htx:
                aggregated_prices[symbol]['HTX'] = prices_htx[symbol]
                deposit_info = await get_withdrawal_deposit_status_htx(session, symbol)
                aggregated_prices[symbol]['HTX_status'] = deposit_info
            
            if symbol in prices_bybit:
                aggregated_prices[symbol]['Bybit'] = prices_bybit[symbol]
                deposit_info = await get_withdrawal_deposit_status_bybit(session, symbol)
                aggregated_prices[symbol]['Bybit_status'] = deposit_info

            if symbol in prices_binance:
                aggregated_prices[symbol]['Binance'] = prices_binance[symbol]
                deposit_info = await get_withdrawal_deposit_status_binance(session, symbol)
                aggregated_prices[symbol]['Binance_status'] = deposit_info

            if symbol in prices_okx:
                aggregated_prices[symbol]['OKX'] = prices_okx[symbol]
                deposit_info = await get_withdrawal_deposit_status_okx(session, symbol)
                aggregated_prices[symbol]['OKX_status'] = deposit_info

            if symbol in prices_mexc:
                aggregated_prices[symbol]['MEXC'] = prices_mexc[symbol]
                deposit_info = await get_withdrawal_deposit_status_mexc(session, symbol)
                aggregated_prices[symbol]['MEXC_status'] = deposit_info

            if symbol in prices_gate:
                aggregated_prices[symbol]['Gate'] = prices_gate[symbol]
                deposit_info = await get_withdrawal_deposit_status_gate(session, symbol)
                aggregated_prices[symbol]['Gate_status'] = deposit_info

            if symbol in prices_bitget:
                aggregated_prices[symbol]['Bitget'] = prices_bitget[symbol]
                deposit_info = await get_withdrawal_deposit_status_bitget(session, symbol)
                aggregated_prices[symbol]['Bitget_status'] = deposit_info
        
        return aggregated_prices

if __name__ == "__main__":
    asyncio.run(aggregate_prices_with_statuses())
