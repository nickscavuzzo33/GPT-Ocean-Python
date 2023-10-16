1. Setup
1.1 Install Ocean
In ocean.py's install.md, follow all steps.

1.2 Install predict-eth
The predict-eth library has a specific error calculation function, and other functions specific to this competition. In the console:

pip install predict-eth
1.3 Install other Python libraries
The READMEs use several numerical & ML libraries. In the console:

pip install python-dateutil==2.8.1 ccxt eth_account matplotlib numpy pandas==1.5.3 prophet requests scikit-learn
1.4 Do Ocean remote setup
In ocean.py's setup-remote.md, follow all steps.

Make sure you're in running in Mumbai!

1.5 Load helper functions
In the Python console:

from predict_eth.helpers import *
2. Get data locally
Here, use whatever data you wish. It can be static data or streams, free or priced, raw data or feature vectors or otherwise. It can be published via Ocean, or not. The main README links to some options.

This demo flow skips getting data because it will generate random predictions (no data needed).

3. Make predictions
3.1 Build a simple AI model
Here, build whatever AI/ML model you want, leveraging the data from the previous step. The main README links to some options.

This demo flow skips building a model because it will generate random predictions (no model needed).

3.2 Run the AI model to make future ETH price predictions
Predictions must be one prediction every 5mins, for a 60min period. The specific times were given above. There are 12 predictions total. The output is a list with 12 items.

Here's an example with random numbers. In the same Python console:

#get predicted ETH values
mean, stddev = 1800, 25.0
pred_vals = list(np.random.normal(loc=mean, scale=stddev, size=(12,)))
3.3 Calculate NMSE
We use normalized mean-squared error (NMSE) as the accuracy measure.

In the same Python console:

# get the time range we want to test for
start_dt = datetime.datetime.utcnow() - datetime.timedelta(minutes=120) #must be >= 60min ago; we use 120
start_dt = round_to_nearest_timeframe(start_dt) # so that times line up
target_uts = target_12_unixtimes(start_dt)
print_datetime_info("target times", target_uts)

# get the actual ETH values at that time
import ccxt
allcex_x = ccxt.binance().fetch_ohlcv('ETH/USDT', '5m')
allcex_uts = [xi[0]/1000 for xi in allcex_x]
allcex_vals = [xi[4] for xi in allcex_x]
print_datetime_info("allcex times", allcex_uts)

cex_vals = filter_to_target_uts(target_uts, allcex_uts, allcex_vals)

# now, we have predicted and actual values. Let's find error, and plot!
nmse = calc_nmse(cex_vals, pred_vals)
print(f"NMSE = {nmse}")
plot_prices(cex_vals, pred_vals)
Keep iterating in step 3 until you're satisfied with accuracy. Then...

4. Publish & share predictions
4.1 Publish & share via Python
In the same Python console:

# Imports
from ocean_lib.ocean import crypto

# Create data NFT
data_nft = ocean.data_nft_factory.create({"from": alice}, 'Data NFT 1', 'DN1')
print(f"Created data NFT with address={data_nft.address}")

# Encrypt predictions with judges' public key, so competitors can't see.
# NOTE: public key is *not* the same thing as address. Using address will not work.
judges_pubkey = '0x3d87bf8bde8c093a16ca5441b5a1053d34a28aca75dc4afffb7a2a513f2a16d2ac41bac68d8fc53058ed4846de25064098bbfaf0e1a5979aeb98028ce69fab6a'
pred_vals_str = str(pred_vals)
pred_vals_str_enc = crypto.asym_encrypt(pred_vals_str, judges_pubkey)

# Store predictions to data NFT, on-chain
data_nft.set_data("predictions", pred_vals_str_enc, {"from": alice})

# Transfer the data NFT to judges, for prediction tamper-resistance
judges_address = '0xA54ABd42b11B7C97538CAD7C6A2820419ddF703E'
token_id = 1
tx = data_nft.safeTransferFrom(alice.address, judges_address, token_id, {"from": alice})

# Ensure the transfer was successful
assert tx.events['Transfer']['to'].lower() == judges_address.lower()

# Print txid, as we'll use it in the next step
print(f"txid from transferring the nft: {tx.txid}")
