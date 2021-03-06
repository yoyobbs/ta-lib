# TA-Lib

这是基于Cython开发的[TA-Lib](http://ta-lib.org) ,替代SWIG.
这是主页的介绍：
> TA-Lib 适用于于交易开发人员
> 财经市场数据技术分析

> * 包含超过150+技术指标，如ADX,MACD,RSI,Stochastic, Bollinger,Bands等
> * Candlestick pattern recognition
> * Open-source API for C/C++, Java, Perl, Python and 100% Managed .NET

The original Python bindings use [SWIG](http://swig.org) which unfortunately
are difficult to install and aren't as efficient as they could be. Therefore
this project uses Cython and Numpy to efficiently and cleanly bind to TA-Lib
-- producing results 2-4 times faster than the SWIG interface.

## Installation
安装方法
通过PYPI安装
```
$ pip install TA-Lib
```
下载源码，执行``setup.py`` 安装
```
$ python setup.py install
```
### 故障排除
安装的时候可能会有以下错误：
```
func.c:256:28: fatal error: ta-lib/ta_libc.h: No such file or directory
compilation terminated.
```
这个意思是找不到底层``TA-Lib``库，需要安装依赖库
### 安装依赖库
python要使用TA-Lib，你需要确保[TA-Lib]已经安装(http://ta-lib.org/hdr_dw.html) 

##### Mac OS X

```
$ brew install ta-lib
```
##### Windows

下载 [ta-lib-0.4.0-msvc.zip](http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-msvc.zip)
并解压到 ``C:\ta-lib``

##### Linux

下载 [ta-lib-0.4.0-src.tar.gz](http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz):
```
$ untar and cd
$ ./configure --prefix=/usr
$ make
$ sudo make install
```

> 如果你编译 ``TA-Lib`` 时使用 ``make -jX`` 会报错，但是是正常的。
> 只需要重新运行``make -jX`` 然后 ``[sudo] make install``.

## API接口

类似 TA-Lib, API 接口提供一个轻量级的TA-Lib指标.

除非指定参数，每个接口根据默认参数返回并输出数组。通常这些接口会有一个初始"lookback"值（在生成输出值前的一个必须值）设置为``NaN``.

为了方便，接口支持两种方式：``numpy.ndarray``和``pandas.Series`` .

以下例子使用了TA-Lib的功能接口

```python
import numpy
import talib

close = numpy.random.random(100)
```

计算一个简单的收市移动平均价

```python
output = talib.SMA(close)
```

计算boll指标（布林线指标），带三指数移动平均线（有关[三指数移动平均线](http://reference.wolfram.com/language/ref/indicator/TripleExponentialMovingAverage.html)解释）

```python
from talib import MA_Type

upper, middle, lower = talib.BBANDS(close, matype=MA_Type.T3)
```

计算收市价上涨趋势，以5日为一周期：

```python
output = talib.MOM(close, timeperiod=5)
```

## 抽象 API
如果你已经熟悉并使用功能API，应该可以轻松自如地使用抽象API

每个接口都需要使用集合（collection）为输入参数，即使是``dict`` 、``numpy.ndarray``、``pandas.Series``、 ``pandas.DataFrame``.如果是``pandas.DataFrame``，则输出以``pandas.DataFrame``命名的列.

例如，输入一个典型的数据"OHLCV":

```python
import numpy as np

# 注意所有ndarrays必须长度一致!
inputs = {
    'open': np.random.random(100),
    'high': np.random.random(100),
    'low': np.random.random(100),
    'close': np.random.random(100),
    'volume': np.random.random(100)
}
```
函数既可以直接输入也可以通过name实例化

```python
from talib import abstract

# directly
sma = abstract.SMA

# or by name
sma = abstract.Function('sma')
```

至此，调用函数与基本的函数API功能相同

```python
from talib.abstract import *

# 使用收盘价 (默认)
output = SMA(inputs, timeperiod=25)

# 使用开盘价
output = SMA(inputs, timeperiod=25, price='open')

# 使用收盘价（默认）
upper, middle, lower = BBANDS(inputs, 20, 2, 2)

# 使用最高价格，最低价，收盘价（默认）
slowk, slowd = STOCH(inputs, 5, 3, 0, 3, 0) # uses high, low, close by default

# 使用最高价格，最低价，开盘价
slowk, slowd = STOCH(inputs, 5, 3, 0, 3, 0, prices=['high', 'low', 'open'])
```
## 支持的指标与函数

通过TA-Lib，我们可以使用全部的TA函数，无论是通过组来排序``list``或者是``dict`` 
(e.g. "Overlap Studies", "Momentum Indicators",
etc):

```python
import talib

# 列表函数
print talib.get_functions()

# 使用字典函数排序
print talib.get_function_groups()
```

### 指标集

* Overlap Studies（重叠研究）
* Momentum Indicators（动量指标）
* Volume Indicators（量指标）
* Volatility Indicators（波动性指标）
* Price Transform（价格）
* Cycle Indicators（循环指标）
* Pattern Recognition（模式识别）
* 

#### Overlap Studies
```
BBANDS               Boll线（Bollinger Bands）
DEMA                 双指数移动平均值（Double Exponential Moving Average）
EMA                  指数移动平均线（Exponential Moving Average）
HT_TRENDLINE         Hilbert Transform - Instantaneous Trendline
KAMA                 Kaufman Adaptive Moving Average
MA                   Moving average
MAMA                 MESA Adaptive Moving Average
MAVP                 Moving average with variable period
MIDPOINT             MidPoint over period
MIDPRICE             Midpoint Price over period
SAR                  Parabolic SAR
SAREXT               Parabolic SAR - Extended
SMA                  Simple Moving Average
T3                   Triple Exponential Moving Average (T3)
TEMA                 Triple Exponential Moving Average
TRIMA                Triangular Moving Average
WMA                  Weighted Moving Average
```

#### Momentum Indicators
```
ADX                  Average Directional Movement Index
ADXR                 Average Directional Movement Index Rating
APO                  Absolute Price Oscillator
AROON                Aroon
AROONOSC             Aroon Oscillator
BOP                  Balance Of Power
CCI                  Commodity Channel Index
CMO                  Chande Momentum Oscillator
DX                   Directional Movement Index
MACD                 Moving Average Convergence/Divergence
MACDEXT              MACD with controllable MA type
MACDFIX              Moving Average Convergence/Divergence Fix 12/26
MFI                  Money Flow Index
MINUS_DI             Minus Directional Indicator
MINUS_DM             Minus Directional Movement
MOM                  Momentum
PLUS_DI              Plus Directional Indicator
PLUS_DM              Plus Directional Movement
PPO                  Percentage Price Oscillator
ROC                  Rate of change : ((price/prevPrice)-1)*100
ROCP                 Rate of change Percentage: (price-prevPrice)/prevPrice
ROCR                 Rate of change ratio: (price/prevPrice)
ROCR100              Rate of change ratio 100 scale: (price/prevPrice)*100
RSI                  Relative Strength Index
STOCH                Stochastic
STOCHF               Stochastic Fast
STOCHRSI             Stochastic Relative Strength Index
TRIX                 1-day Rate-Of-Change (ROC) of a Triple Smooth EMA
ULTOSC               Ultimate Oscillator
WILLR                Williams' %R
```

#### Volume Indicators
```
AD                   Chaikin A/D Line
ADOSC                Chaikin A/D Oscillator
OBV                  On Balance Volume
```

#### Cycle Indicators
```
HT_DCPERIOD          Hilbert Transform - Dominant Cycle Period
HT_DCPHASE           Hilbert Transform - Dominant Cycle Phase
HT_PHASOR            Hilbert Transform - Phasor Components
HT_SINE              Hilbert Transform - SineWave
HT_TRENDMODE         Hilbert Transform - Trend vs Cycle Mode
```

#### Price Transform
```
AVGPRICE             Average Price
MEDPRICE             Median Price
TYPPRICE             Typical Price
WCLPRICE             Weighted Close Price
```

#### Volatility Indicators
```
ATR                  Average True Range
NATR                 Normalized Average True Range
TRANGE               True Range
```

#### Pattern Recognition
```
CDL2CROWS            Two Crows
CDL3BLACKCROWS       Three Black Crows
CDL3INSIDE           Three Inside Up/Down
CDL3LINESTRIKE       Three-Line Strike
CDL3OUTSIDE          Three Outside Up/Down
CDL3STARSINSOUTH     Three Stars In The South
CDL3WHITESOLDIERS    Three Advancing White Soldiers
CDLABANDONEDBABY     Abandoned Baby
CDLADVANCEBLOCK      Advance Block
CDLBELTHOLD          Belt-hold
CDLBREAKAWAY         Breakaway
CDLCLOSINGMARUBOZU   Closing Marubozu
CDLCONCEALBABYSWALL  Concealing Baby Swallow
CDLCOUNTERATTACK     Counterattack
CDLDARKCLOUDCOVER    Dark Cloud Cover
CDLDOJI              Doji
CDLDOJISTAR          Doji Star
CDLDRAGONFLYDOJI     Dragonfly Doji
CDLENGULFING         Engulfing Pattern
CDLEVENINGDOJISTAR   Evening Doji Star
CDLEVENINGSTAR       Evening Star
CDLGAPSIDESIDEWHITE  Up/Down-gap side-by-side white lines
CDLGRAVESTONEDOJI    Gravestone Doji
CDLHAMMER            Hammer
CDLHANGINGMAN        Hanging Man
CDLHARAMI            Harami Pattern
CDLHARAMICROSS       Harami Cross Pattern
CDLHIGHWAVE          High-Wave Candle
CDLHIKKAKE           Hikkake Pattern
CDLHIKKAKEMOD        Modified Hikkake Pattern
CDLHOMINGPIGEON      Homing Pigeon
CDLIDENTICAL3CROWS   Identical Three Crows
CDLINNECK            In-Neck Pattern
CDLINVERTEDHAMMER    Inverted Hammer
CDLKICKING           Kicking
CDLKICKINGBYLENGTH   Kicking - bull/bear determined by the longer marubozu
CDLLADDERBOTTOM      Ladder Bottom
CDLLONGLEGGEDDOJI    Long Legged Doji
CDLLONGLINE          Long Line Candle
CDLMARUBOZU          Marubozu
CDLMATCHINGLOW       Matching Low
CDLMATHOLD           Mat Hold
CDLMORNINGDOJISTAR   Morning Doji Star
CDLMORNINGSTAR       Morning Star
CDLONNECK            On-Neck Pattern
CDLPIERCING          Piercing Pattern
CDLRICKSHAWMAN       Rickshaw Man
CDLRISEFALL3METHODS  Rising/Falling Three Methods
CDLSEPARATINGLINES   Separating Lines
CDLSHOOTINGSTAR      Shooting Star
CDLSHORTLINE         Short Line Candle
CDLSPINNINGTOP       Spinning Top
CDLSTALLEDPATTERN    Stalled Pattern
CDLSTICKSANDWICH     Stick Sandwich
CDLTAKURI            Takuri (Dragonfly Doji with very long lower shadow)
CDLTASUKIGAP         Tasuki Gap
CDLTHRUSTING         Thrusting Pattern
CDLTRISTAR           Tristar Pattern
CDLUNIQUE3RIVER      Unique 3 River
CDLUPSIDEGAP2CROWS   Upside Gap Two Crows
CDLXSIDEGAP3METHODS  Upside/Downside Gap Three Methods
```

#------------------------------------------------------------

This is a Python wrapper for [TA-LIB](http://ta-lib.org) based on Cython
instead of SWIG. From the homepage:

> TA-Lib is widely used by trading software developers requiring to perform
> technical analysis of financial market data.

> * Includes 150+ indicators such as ADX, MACD, RSI, Stochastic, Bollinger
>   Bands, etc.
> * Candlestick pattern recognition
> * Open-source API for C/C++, Java, Perl, Python and 100% Managed .NET

The original Python bindings use [SWIG](http://swig.org) which unfortunately
are difficult to install and aren't as efficient as they could be. Therefore
this project uses Cython and Numpy to efficiently and cleanly bind to TA-Lib
-- producing results 2-4 times faster than the SWIG interface.
### 英文原文
You can install from PyPI:

```
$ pip install TA-Lib
```

Or checkout the sources and run ``setup.py`` yourself:

```
$ python setup.py install
```

### Troubleshooting

Sometimes installation will produce build errors like this:

```
func.c:256:28: fatal error: ta-lib/ta_libc.h: No such file or directory
compilation terminated.
```

This typically means that it can't find the underlying ``TA-Lib`` library,
a dependency which needs to be installed.

### Dependencies

To use TA-Lib for python, you need to have the
[TA-Lib](http://ta-lib.org/hdr_dw.html) already installed:

##### Mac OS X

```
$ brew install ta-lib
```

##### Windows

Download [ta-lib-0.4.0-msvc.zip](http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-msvc.zip)
and unzip to ``C:\ta-lib``

##### Linux

Download [ta-lib-0.4.0-src.tar.gz](http://prdownloads.sourceforge.net/ta-lib/ta-lib-0.4.0-src.tar.gz) and:
```
$ untar and cd
$ ./configure --prefix=/usr
$ make
$ sudo make install
```

> If you build ``TA-Lib`` using ``make -jX`` it will fail but that's OK!
> Simply rerun ``make -jX`` followed by ``[sudo] make install``.

## Function API

Similar to TA-Lib, the Function API provides a lightweight wrapper of the
exposed TA-Lib indicators.

Each function returns an output array and have default values for their
parameters, unless specified as keyword arguments. Typically, these functions
will have an initial "lookback" period (a required number of observations
before an output is generated) set to ``NaN``.

For convenience, the Function API supports both ``numpy.ndarray`` and
``pandas.Series`` types.

All of the following examples use the Function API:

```python
import numpy
import talib

close = numpy.random.random(100)
```

Calculate a simple moving average of the close prices:

```python
output = talib.SMA(close)
```

Calculating bollinger bands, with triple exponential moving average:

```python
from talib import MA_Type

upper, middle, lower = talib.BBANDS(close, matype=MA_Type.T3)
```

Calculating momentum of the close prices, with a time period of 5:

```python
output = talib.MOM(close, timeperiod=5)
```

## Abstract API

If you're already familiar with using the function API, you should feel right
at home using the Abstract API.

Every function takes a collection of named inputs, either a ``dict`` of
``numpy.ndarray`` or ``pandas.Series``, or a ``pandas.DataFrame``. If a
``pandas.DataFrame`` is provided, the output is returned as a
``pandas.DataFrame`` with named output columns.

For example, inputs could be provided for the typical "OHLCV" data:

```python
import numpy as np

# note that all ndarrays must be the same length!
inputs = {
    'open': np.random.random(100),
    'high': np.random.random(100),
    'low': np.random.random(100),
    'close': np.random.random(100),
    'volume': np.random.random(100)
}
```

Functions can either be imported directly or instantiated by name:

```python
from talib import abstract

# directly
sma = abstract.SMA

# or by name
sma = abstract.Function('sma')
```

From there, calling functions is basically the same as the function API:

```python
from talib.abstract import *

# uses close prices (default)
output = SMA(inputs, timeperiod=25)

# uses open prices
output = SMA(inputs, timeperiod=25, price='open')

# uses close prices (default)
upper, middle, lower = BBANDS(inputs, 20, 2, 2)

# uses high, low, close (default)
slowk, slowd = STOCH(inputs, 5, 3, 0, 3, 0) # uses high, low, close by default

# uses high, low, open instead
slowk, slowd = STOCH(inputs, 5, 3, 0, 3, 0, prices=['high', 'low', 'open'])
```

## Supported Indicators and Functions

We can show all the TA functions supported by TA-Lib, either as a ``list`` or
as a ``dict`` sorted by group (e.g. "Overlap Studies", "Momentum Indicators",
etc):

```python
import talib

# list of functions
print talib.get_functions()

# dict of functions by group
print talib.get_function_groups()
```

### Indicator Groups

* Overlap Studies
* Momentum Indicators
* Volume Indicators
* Volatility Indicators
* Price Transform
* Cycle Indicators
* Pattern Recognition

#### Overlap Studies
```
BBANDS               Bollinger Bands
DEMA                 Double Exponential Moving Average
EMA                  Exponential Moving Average
HT_TRENDLINE         Hilbert Transform - Instantaneous Trendline
KAMA                 Kaufman Adaptive Moving Average
MA                   Moving average
MAMA                 MESA Adaptive Moving Average
MAVP                 Moving average with variable period
MIDPOINT             MidPoint over period
MIDPRICE             Midpoint Price over period
SAR                  Parabolic SAR
SAREXT               Parabolic SAR - Extended
SMA                  Simple Moving Average
T3                   Triple Exponential Moving Average (T3)
TEMA                 Triple Exponential Moving Average
TRIMA                Triangular Moving Average
WMA                  Weighted Moving Average
```

#### Momentum Indicators
```
ADX                  Average Directional Movement Index
ADXR                 Average Directional Movement Index Rating
APO                  Absolute Price Oscillator
AROON                Aroon
AROONOSC             Aroon Oscillator
BOP                  Balance Of Power
CCI                  Commodity Channel Index
CMO                  Chande Momentum Oscillator
DX                   Directional Movement Index
MACD                 Moving Average Convergence/Divergence
MACDEXT              MACD with controllable MA type
MACDFIX              Moving Average Convergence/Divergence Fix 12/26
MFI                  Money Flow Index
MINUS_DI             Minus Directional Indicator
MINUS_DM             Minus Directional Movement
MOM                  Momentum
PLUS_DI              Plus Directional Indicator
PLUS_DM              Plus Directional Movement
PPO                  Percentage Price Oscillator
ROC                  Rate of change : ((price/prevPrice)-1)*100
ROCP                 Rate of change Percentage: (price-prevPrice)/prevPrice
ROCR                 Rate of change ratio: (price/prevPrice)
ROCR100              Rate of change ratio 100 scale: (price/prevPrice)*100
RSI                  Relative Strength Index
STOCH                Stochastic
STOCHF               Stochastic Fast
STOCHRSI             Stochastic Relative Strength Index
TRIX                 1-day Rate-Of-Change (ROC) of a Triple Smooth EMA
ULTOSC               Ultimate Oscillator
WILLR                Williams' %R
```

#### Volume Indicators
```
AD                   Chaikin A/D Line
ADOSC                Chaikin A/D Oscillator
OBV                  On Balance Volume
```

#### Cycle Indicators
```
HT_DCPERIOD          Hilbert Transform - Dominant Cycle Period
HT_DCPHASE           Hilbert Transform - Dominant Cycle Phase
HT_PHASOR            Hilbert Transform - Phasor Components
HT_SINE              Hilbert Transform - SineWave
HT_TRENDMODE         Hilbert Transform - Trend vs Cycle Mode
```

#### Price Transform
```
AVGPRICE             Average Price
MEDPRICE             Median Price
TYPPRICE             Typical Price
WCLPRICE             Weighted Close Price
```

#### Volatility Indicators
```
ATR                  Average True Range
NATR                 Normalized Average True Range
TRANGE               True Range
```

#### Pattern Recognition
```
CDL2CROWS            Two Crows
CDL3BLACKCROWS       Three Black Crows
CDL3INSIDE           Three Inside Up/Down
CDL3LINESTRIKE       Three-Line Strike
CDL3OUTSIDE          Three Outside Up/Down
CDL3STARSINSOUTH     Three Stars In The South
CDL3WHITESOLDIERS    Three Advancing White Soldiers
CDLABANDONEDBABY     Abandoned Baby
CDLADVANCEBLOCK      Advance Block
CDLBELTHOLD          Belt-hold
CDLBREAKAWAY         Breakaway
CDLCLOSINGMARUBOZU   Closing Marubozu
CDLCONCEALBABYSWALL  Concealing Baby Swallow
CDLCOUNTERATTACK     Counterattack
CDLDARKCLOUDCOVER    Dark Cloud Cover
CDLDOJI              Doji
CDLDOJISTAR          Doji Star
CDLDRAGONFLYDOJI     Dragonfly Doji
CDLENGULFING         Engulfing Pattern
CDLEVENINGDOJISTAR   Evening Doji Star
CDLEVENINGSTAR       Evening Star
CDLGAPSIDESIDEWHITE  Up/Down-gap side-by-side white lines
CDLGRAVESTONEDOJI    Gravestone Doji
CDLHAMMER            Hammer
CDLHANGINGMAN        Hanging Man
CDLHARAMI            Harami Pattern
CDLHARAMICROSS       Harami Cross Pattern
CDLHIGHWAVE          High-Wave Candle
CDLHIKKAKE           Hikkake Pattern
CDLHIKKAKEMOD        Modified Hikkake Pattern
CDLHOMINGPIGEON      Homing Pigeon
CDLIDENTICAL3CROWS   Identical Three Crows
CDLINNECK            In-Neck Pattern
CDLINVERTEDHAMMER    Inverted Hammer
CDLKICKING           Kicking
CDLKICKINGBYLENGTH   Kicking - bull/bear determined by the longer marubozu
CDLLADDERBOTTOM      Ladder Bottom
CDLLONGLEGGEDDOJI    Long Legged Doji
CDLLONGLINE          Long Line Candle
CDLMARUBOZU          Marubozu
CDLMATCHINGLOW       Matching Low
CDLMATHOLD           Mat Hold
CDLMORNINGDOJISTAR   Morning Doji Star
CDLMORNINGSTAR       Morning Star
CDLONNECK            On-Neck Pattern
CDLPIERCING          Piercing Pattern
CDLRICKSHAWMAN       Rickshaw Man
CDLRISEFALL3METHODS  Rising/Falling Three Methods
CDLSEPARATINGLINES   Separating Lines
CDLSHOOTINGSTAR      Shooting Star
CDLSHORTLINE         Short Line Candle
CDLSPINNINGTOP       Spinning Top
CDLSTALLEDPATTERN    Stalled Pattern
CDLSTICKSANDWICH     Stick Sandwich
CDLTAKURI            Takuri (Dragonfly Doji with very long lower shadow)
CDLTASUKIGAP         Tasuki Gap
CDLTHRUSTING         Thrusting Pattern
CDLTRISTAR           Tristar Pattern
CDLUNIQUE3RIVER      Unique 3 River
CDLUPSIDEGAP2CROWS   Upside Gap Two Crows
CDLXSIDEGAP3METHODS  Upside/Downside Gap Three Methods
```
