# traderJS v2
A spinoff of our original module written in Python and targeted at Tradestation
## Disclaimer
Documentation will be provided in the event that we receive a bunch of requests. In the meantime, just read over the code before you use it- the majority of the functions are self explanatory. In order to fully understand what is going on here and how to implement this into your own program, please read up on Tradestation's [DOCUMENTATION](https://api.tradestation.com/docs/specification). 
## Core functions
In this section I will summarize the core parts of this module to help you get up and running a little bit faster.
### Access token refresh
This must be called in order to get a new access token from your refresh token. I am not going to cover how to get a refresh token, as that is outside of the scope of this program, however, a quick google search will get you setup with one in a matter of minutes. I advise using this in a 2nd thread and using the expires_in as a way to ensure that the token never expires. The expires_in variable is expressed in seconds. In the example below you would place the refresh_access_token() function in a separate thread and the program would take care of the refreshing for you.
```
def get_new_access_token(refresh_token, client_id, client_secret):
    token_url = 'https://signin.tradestation.com/oauth/token'
    headers = {'content-type': 'application/x-www-form-urlencoded'}

    payload = {
        'grant_type': 'refresh_token',
        'client_id': client_id,
        'client_secret': client_secret,
        'refresh_token': refresh_token
    }

    response = requests.post(token_url, data=payload, headers=headers)

    if response.status_code == 200:
        data = response.json()
        access_token = data.get('access_token')
        expires_in = data.get('expires_in')
        return access_token, expires_in
    else:
        raise Exception(f"Failed to get a new access token. Response: {response.text}")

def refresh_access_token():
    with open(r"C:\\Users\\INSERT A VALID PATH HERE\\private.json", 'r') as file:
        private_data = json.load(file)

    with open(r"C:\\Users\\INSERT A VALID PATH HERE\\refreshToken.json", 'r') as file:
        refresh = json.load(file)

    client_id = private_data['client_id']
    client_secret = private_data.get('client_secret', '')
    refresh_token = refresh['refresh']

    on = True

    while on == True:
        try:
            access_token, expires_in = get_new_access_token(refresh_token, client_id, client_secret)
            data = {"accessToken": access_token}
            with open(r"C:\\Users\\INSERT A VALID PATH HERE\\access_token.json", "w") as json_file:
                json.dump(data, json_file)
            print(f"New access token aquired. Expires in: {expires_in} seconds")
            time.sleep(expires_in - 60)
        except KeyboardInterrupt:
            on = False
            print_red_text("Shutting down the refresh loop...")
            sys.exit(0)
        except Exception as e:
            print(f"Error: {e}")
            break
```
### Ordering
Being able to send trades out is kind of important. I won't explain this example- just read through it. This assunes that you have the access token auto refresh taken care of somewhere else (a separate thread).
```
def print_blue_text(text):
    blue_text = "\033[34m" + text + "\033[0m"
    print(blue_text)

def print_red_text(text):
    red_text = "\033[31m" + text + "\033[0m"
    print(red_text)

def load_access_token():
    path = 'C:\\Users\\admin\\Desktop\\Million Dollar Business\\argonaut\\access_token.json'

    with open(f'{path}') as file:
        data = json.load(file)
        return data['accessToken']

def placeTrade(quantityEnter, quantityEnter2, direction, tsdirec, tspoints):
    access_token = load_access_token()
    account_id = 'SIM2504240F'
    symbol = 'MNQU23'
    quantity = quantityEnter
    quantity2 = quantityEnter2
    order_type = 'Market'
    action = direction
    actionStop = tsdirec
    points = tspoints

    url = 'https://sim-api.tradestation.com/v3/orderexecution/orders'
    headers = {
        'content-type': 'application/json',
        'Authorization': f'Bearer {access_token}',
    }
    data = {
        "AccountID": f"{account_id}",
        "TimeInForce": {
            "Duration": "DAY"
        },
        "Quantity": f"{quantity}",
        "OrderType": f"{order_type}",
        "Symbol": f"{symbol}",
        "TradeAction": f"{action}",
        "Route": "Intelligent",
        "OSOs": [
            {
                "Type": "NORMAL",
                "Orders": [
                    {
                        "AccountID": f"{account_id}",
                        "TimeInForce": {
                            "Duration": "DAY"
                        },
                        "Quantity": f"{quantity2}",
                        "OrderType": "StopMarket",
                        "Symbol": f"{symbol}",
                        "TradeAction": f"{actionStop}",
                        "Route": "Intelligent",
                        "AdvancedOptions": {
                            "TrailingStop": {
                                "Amount": f"{points}"
                            }
                        }
                    }
                ]
            }
        ]
    } 

    try:
        response = requests.post(url, headers=headers, json=data)
        response_data = response.json()
        print(response_data)
        print(f'Order for {direction} {symbol} placed successfully.')
        try:
            first_order = response_data['Orders'][0]
            first_order_id = first_order['OrderID']
            print(f'Order ID for the trailingstop is {first_order_id}')
            path = 'C:\\Users\\admin\\Desktop\\Million Dollar Business\\argonaut\\ts_orderid.txt'
            with open(f'{path}', 'w') as f:
                f.write(first_order_id)
        except Exception as e:
            print_red_text("\n------ WARNING ------")
            print_red_text("Stop ID logging failed.")
            print_red_text("---- END WARNING ----")
    except Exception as e:
        print_red_text("\n------ WARNING ------")
        print_red_text("Order placement loop failed.")
        print_red_text("---- END WARNING ----")

placeTrade("10", "10", "SELL", "BUY", "20")
```
### Other features
This is just displaying the basics of what this module does. Once again, if you want to understand it and be able to successfully implement it utilizing all of the available features, read the code.
