via https://pypi.org/project/bitfinex-v2/
via  https://github.com/ohenrik/bitfinex

wss客户端
python虚拟环境 linux

create a new environment

mkdir btenv

cd btenv

python3 -m venv env

source env/bin/activate

pip3 install git+https://github.com/ohenrik/bitfinex.git

try and use the wss example from my pull request, it will work.

https://github.com/dantimofte/bitfinex/blob/master/examples/wss_example.py

!!!! be carefull to comment out the neworder command !!!

windows 提示 

ValueError: Invalid format string

或

/client.py", line 308, in new_order_op flags": sum(flags)

TypeError: unsupported operand type(s) for +: 'int' and 'str'

I edit C:\pytest1\Lib\site-packages\bitfinex\websockets\wss_utils.py UtcNow(), The wss_example.py is OK on my Windows 10 python 36

#windows 10 py36 add

import time

def UtcNow():

#windows 10 py36

now = time.time()

#linux
#now = datetime.datetime.utcnow()

#return int(float(now.strftime("%s.%f"))*10000000)

return int(now * 10000000)

delete C:\pytest1\Lib\site-packages\bitfinex\websockets_pycache_\wss_utils.cpython-36.pyc

(pytest1) C:\pytest1>python wss_example.py


=============

增加对stop_limit 的支持
修改client.py
new_order_op
new_order

赋值这2个函数
 def new_order_op_stop_limit(self, order_type, pair, amount, price, stop_limit_price,hidden=0, flags=None):

        #add  'price_aux_limit':stop_limit_price

        flags = flags or []
        client_order_id = wss_utils.UtcNow()
        return {
            'cid': client_order_id,
            'type': order_type,
            'symbol': wss_utils.order_pair(pair),
            'amount': amount,
            'price': price,
            'price_aux_limit':stop_limit_price,  
            'hidden': hidden,
            "flags": sum(flags)
        }
        
    #上面的  'price_aux_limit':stop_limit_price,   是新增加的    
  def new_order_stop_limit(self, order_type, pair, amount, price, stop_limit_price, hidden=0, flags=list()):


        operation = self.new_order_op_stop_limit(order_type, pair, amount, price, stop_limit_price, 0, flags)
        data = [0,wss_utils.get_notification_code('order new'),None,operation]
        payload = json.dumps(data, ensure_ascii = False).encode('utf8')
        self.factories["auth"].protocol_instance.sendMessage(payload, isBinary=False)
        return operation["cid"]        
        
        
使用方法
如果当前价格是 5元, 

                exch_order = self.mywss.new_order_stop_limit("STOP LIMIT", "tEOSUSD", "3", "6","6.2")
                当最新价格高于6元时候, 开一个6.2元限价的买多单, 作用你想想....
                
                
                exch_order = self.mywss.new_order_stop_limit("STOP LIMIT", "tEOSUSD", "-3", "4","3.9")
                当最新价格低于4元时候, 开一个3.9元限价的卖空单, 作用你想想....
                


1 提示错误 ImportError: DLL load failed: 操作系统无法运行 %1。

如果使用Anaconda3 需要复制2个dll文件 libeay32.dll ssleay32.dll 到  Anaconda3\Lib\site-packages\cryptography\hazmat\bindings 目录下面, 文件在 Anaconda3\pkgs\openssl-1.0.2o-h8ea7d77_0\Library\bin

Copy both files from Anaconda3\pkgs\openssl-1.0.2o-h8ea7d77_0\Library\bin into Anaconda3\Lib\site-packages\cryptography\hazmat\bindings, and it works

更多: https://github.com/pyca/cryptography/issues/4011

hello
me too i meet this problem under windows 10 , after many search on many websites . i found this solution :
download this : https://github.com/python/cpython-bin-deps/tree/openssl-bin-1.0.2k
zip the file and copy the folder (amd or win ) in your sys path : C:\Windows\SysWOW64
and voila every thing works fine


# Bitfinex Python Client

**Continuation of**: https://github.com/scottjbarr/bitfinex

A Python client for the Bitfinex API v1 and v2 + websockets for v2.

## Installation

    pip install bitfinex-v2

## Setup

Install the libs

    pip install -r ./requirements.txt

## Documentation

Full documentation coming soon. Please refer to the code to see function
names etc. for now.


## Usage REST API (v2)

    from bitfinex import ClientV2 as Client
    client = Client(
        os.environ.get('BITFINEX_KEY'),
        os.environ.get('BITFINEX_SECRET')
    )

    client.ticker('tBTCUSD')
    >>>>  [0.00011989,
          224267.1563644,
          0.00012015,
          265388.46405117,
          -8.12e-06,
          -0.0634,
          0.00012,
          3206446.72556849,
          0.00012812,
          0.00011545
    ]

## Usage Websockets API (v2)

The documentation for bitfinex websockets are somewhat confusing
So expect to spend some time figuring out just what is actually returned
through the authenticated channel.

    import os
    key = os.environ.get("API_KEY")
    secret = os.environ.get("API_SECRET")
    from bitfinex.websockets import WssClient
    bm = WssClient(key, secret)

    # Authenticate and listen to account feedback
    # print is the callback method. You probably want to do something else.
    bm.authenticate(print)

    # Subscribe to candles (starts with a set of completed candles, then updates with last completed + updated uncompleted candle)
    bm.subscribe_to_candles("BTCUSD", "1m", print)
    bm.start()

    # Send new order
    bm.new_order(order_type="EXCHANGE LIMIT", pair="BTCUSD", amount="0.1", price="1", hidden=0)

For sending messages I have only implemented new_order and cancel_order.
I will add documentation for this later, for now take a look at the source code.


## Usage REST API (v1)

    from bitfinex import ClientV1 as Client
    client = Client(
      os.environ.get('BITFINEX_KEY'),
      os.environ.get('BITFINEX_SECRET')
    )

    client.ticker('btcusd')
    >>>>  { "mid": "562.56495",
            "bid": "562.15",
            "ask": "562.9799",
            "last_price": "562.25",
            "volume": "7842.11542563",
            "timestamp": "1395552658.339936691"
          }


## Compatibility

This code has been tested on

- Python 3.6

But the REST library will probably work on python 2.7 as well.

I haven’t tested the Websocket library on 2.7

## Tests

When continuing this project (from scottjbarr) I decided to use pytests.

so this should start your tests:

    pytest -v

However... Due to the fact that Mocket (python-mocket) crashes whenever
pyopenssl is installed. Tests related to the rest library does now work unless
you uninstall pyopenssl.

## TODO

- Add the rest of Websocket messages and channels.
- Possible parsing improvement for v2 responses.
- Implement all API calls that Bitfinex make available (v1).

## Contributing

Contributions are welcome and i will do my best to merge PR quickly.

Here are some guidelines that makes everything easier for everybody:

1. Fork it.
1. Create a feature branch containing only your fix or feature.
1. Preferably add/update tests. Features or fixes that don't have good tests won't be accepted before someone adds them (mostly...).
1. Create a pull request.


## References

- This project is a continuation of: https://github.com/scottjbarr/bitfinex
- [https://www.bitfinex.com/pages/api](https://www.bitfinex.com/pages/api)
- [https://community.bitfinex.com/showwiki.php?title=Sample+API+Code](https://community.bitfinex.com/showwiki.php?title=Sample+API+Code)
- [https://gist.github.com/jordanbaucke/5812039](https://gist.github.com/jordanbaucke/5812039)

## Licence

The MIT License (MIT)

Copyright (c) 2014-2015 Scott Barr
^ Original project created by this guy.

See [LICENSE.md](LICENSE.md)
