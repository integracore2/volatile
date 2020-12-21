<table>
<tr>
<td width=170>
  
![b1d3115b-97e1-4703-b382-49fc786b9e19_200x200](https://user-images.githubusercontent.com/32386694/100524005-e02f4280-31b4-11eb-9765-a53c138929d9.png)

</td>
<td>

# Volatile 
### Your day-to-day trading companion.

The word "volatile" comes from the Latin *volatilis*, meaning "having wings" or "able to fly". With time, the financial market adopted it to describe asset price variability over time. Here, Volatile becomes a "trading companion", designed to help you every day to make unemotional, algorithmic-based, trading decisions.

</td>
</tr>
</table>

If you expect Volatile to predict the unpredictable, you are in the wrong place. Be reasonable: this is a swing trading software, runnable on your laptop, aimed to quickly discover out-of-trend opportunities by comparing current stock prices to their projections in a few days. If the current price is much lower than its future projection, perhaps it is a good opportunity to buy; vice versa, if it is much higher, perhaps it is a good moment to sell. This does neither mean the projection will be necessarily met, nor that you will make a short-term profit for every single transaction you make. Anything could happen. However, running Volatile on a daily basis will put you in condition to very quickly survey the market, find good opportunities and base your trading decisions on models, algorithms and data. 

### What to expect and how to use
Volatiles estimates stock trends, predict short-term future prices, then ranks and rates. All you need to do to run Volatile is to open your terminal and type
```ruby
python volatile.py
```
Volatile will automatically analyse the list of stock symbols saved in `symbols_list.txt`. This should neither be considered as representative nor complete list of stocks; feel free to update it as you please (do not worry if by chance you enter a symbol twice). Mind that it can take a while to access information of stock symbols that are either not in the list or that you pass for the first time. For this reason, relevant stock information is stored in `stock_info.csv` and will be fast to access from the second time onwards.

You can also directly ask Volatile to analyse a list of stocks that you are interested in. Mind that, if this list is not fairly large, Volatile will not have enough exposition to the market to provide accurate results. Anyway, as a didactical example, suppose that you want Volatile to analyse the following stock symbols: GOOGL, AAPL, MSFT, NFLX, FB, IBM, SAP, DELL, BABA and AMZN. You can do so by typing
```ruby
python volatile.py -s GOOGL AAPL MSFT NFLX FB IBM SAP DELL BABA AMZN
```
When the run is complete, a prediction table like the following will appear printed on your shell:

<img width="1356" alt="Screenshot 2020-12-16 at 00 15 23" src="https://user-images.githubusercontent.com/32386694/102284107-e9017180-3f33-11eb-8ee0-c3b621dbab96.png">

For each symbol, the table tells you its sector and industry, then the last available observed price, the predicted closing price at the next trading day, its standard deviation and finally a rating. Possible ratings are HIGHLY ABOVE TREND, ABOVE TREND, ALONG TREND, BELOW TREND and HIGHLY BELOW TREND. Symbols appear in the table ranked from the furthest above to the furthest below their respective trends. Ranking and rating are derived from a score metric that compares the predicted closing price in 5 trading days (usually this corresponds to the price in one week) to the last available observed price, scaling by the standard deviation of the prediction; see the technical part below for more details. The prediction table can be saved in the current directory as `prediction_table.csv` by adding the following flag to the command above: `--save-table`.

In the current directory, several estimation plots will appear. `stock_estimation.png` is a visualisation of stock prices and their estimations over the last year, together with a notion of uncertainty. Only stocks rated either above or below their trends will be plotted, ranked as in the prediction table. Notice how the estimation crucially attempts to reproduce the trend of a stock but not to learn its noise. The uncertainty, on the other hand, depends on the stock volatility; the smaller the volatility, the more confident we are about our estimates, the more a sudden shift from the trend will be regarded as significant. You can use this plot as a sanity check that the estimation procedure agrees with your intuition. Make sure to glance at it before any transaction.
![stock_estimation](https://user-images.githubusercontent.com/32386694/102284204-0f271180-3f34-11eb-99de-e47424ec1716.png)

 `sector_estimation.png` and `industry_estimation.png` are plots that help you visualise estimated sector and industry performances. A sector estimate can be though as the average behaviour of its belonging industries, which in turn should be regarded as the average behaviour of its belonging stocks. Both sectors and industries are ranked in alphabetical order. If, like in this case, you only provide Volatile a few symbols, you cannot expect sector and industry information to be accurate. However, when a larger amount of symbols is provided, these plots are a good way to quickly check how sector and industry of the stock that you intend to buy or sell are doing.
![sector_estimation](https://user-images.githubusercontent.com/32386694/102284136-f28ad980-3f33-11eb-8326-2d4f626b2131.png)
![industry_estimation](https://user-images.githubusercontent.com/32386694/102284133-f0287f80-3f33-11eb-9e50-6a7fd5d4697b.png)

Finally,  `market_estimation.png` shows the overall estimated market trend. This can be considered as the average of the sector estimates. Like for sectors and industries, this estimate cannot be accurate if only a few stocks are provided. Use this plot to immediately know in what phase the stock market currently is.
![market_estimation](https://user-images.githubusercontent.com/32386694/102284145-f4ed3380-3f33-11eb-9587-4e2fcf8ac23f.png)
If you do not want plots to be saved in the current directory, you can disable them by adding the flag `--no-plots`.

### How to install
Installing Volatile follows a standard procedure. First, open a terminal and go to the directory where you intend to install Volatile. On a Mac or Linux, you can do so by typing
```ruby
cd path/to/your/directory
```
If you are fine installing Volatile in your home directory, instead of the command before you can just type ``cd``. Then, download Volatile from Github and get in its main directory by typing
```ruby
git clone https://github.com/gianlucadetommaso/volatile.git
cd volatile
```
We recommend to activate a virtual environment. Type
```ruby
pip install virtualenv
virtualenv venv 
source venv/bin/activate
```
Now that you are in your virtual environment, install the dependencies:
```ruby
pip install tensorflow-cpu tensorflow-probability matplotlib yfinance
```
Alternatively, you can also use the requirements file; type `pip install -r requirements.txt`. 

**Important**: Tensorflow is currently supported only up to Python 3.8, not yet Python 3.9; make sure to activate the virtual environment with the right Python version.

Done! You're all set to use Volatile. 

### Behind the scenes (technical)
Volatile adopts a Bayesian hierarchical model based on price, sector and industry information, estimating log-price via polynomials in time. 

Denote <a href="https://www.codecogs.com/eqnedit.php?latex=\tau_t=t/T" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\tau_t=t/T" title="\tau_t=t/T" /></a> to represent times at which observations arrive. <a href="https://www.codecogs.com/eqnedit.php?latex=T" target="_blank"><img src="https://latex.codecogs.com/gif.latex?T" title="T" /></a> corresponds to the number of days in the training dataset, which is taken to be the last one year of data.

Furthermore, denote <a href="https://www.codecogs.com/eqnedit.php?latex=\sigma_j=(D&plus;1-j)/(D&plus;1),\text{&space;for&space;}j=0,\dots,D" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\sigma_j=(D&plus;1-j)/(D&plus;1),\text{&space;for&space;}j=0,\dots,D" title="\sigma_j=(D+1-j)/(D+1),\text{ for }j=0,\dots,D" /></a> to be prior scale parameters associated to the j-th order of a polynomial with degree <a href="https://www.codecogs.com/eqnedit.php?latex=D" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D" title="D" /></a>. Decreasing the scales as <a href="https://www.codecogs.com/eqnedit.php?latex=j" target="_blank"><img src="https://latex.codecogs.com/gif.latex?j" title="j" /></a> increases penalises deviation from zero of higher-order parameters, thereby encouraging simpler models. We will describe below how to select the model complexity <a href="https://www.codecogs.com/eqnedit.php?latex=D" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D" title="D" /></a>.

We write:
- <a href="https://www.codecogs.com/eqnedit.php?latex=\text{sec}(\ell)=k,\text{&space;for&space;}\ell=1,\dots,&space;L\text{&space;and&space;}k=1\dots,K" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\text{sec}(\ell)=k,\text{&space;for&space;}\ell=1,\dots,&space;L\text{&space;and&space;}k=1\dots,K" title="\text{sec}(\ell)=k,\text{ for }\ell=1,\dots, L\text{ and }k=1\dots,K" /></a> to indicate that an industry <a href="https://www.codecogs.com/eqnedit.php?latex=\ell" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\ell" title="\ell" /></a> belongs to a sector <a href="https://www.codecogs.com/eqnedit.php?latex=k" target="_blank"><img src="https://latex.codecogs.com/gif.latex?k" title="k" /></a>, where <a href="https://www.codecogs.com/eqnedit.php?latex=L" target="_blank"><img src="https://latex.codecogs.com/gif.latex?L" title="L" /></a> is the number of industries and <a href="https://www.codecogs.com/eqnedit.php?latex=K" target="_blank"><img src="https://latex.codecogs.com/gif.latex?K" title="K" /></a> the number of sectors;
- <a href="https://www.codecogs.com/eqnedit.php?latex=\text{ind}(i)=\ell,\text{&space;for&space;}i=1,\dots,&space;N\text{&space;and&space;}\ell=1\dots,L" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\text{ind}(i)=\ell,\text{&space;for&space;}i=1,\dots,&space;N\text{&space;and&space;}\ell=1\dots,L" title="\text{ind}(i)=\ell,\text{ for }i=1,\dots, N\text{ and }\ell=1\dots,L" /></a> to indicate that a stock <a href="https://www.codecogs.com/eqnedit.php?latex=i" target="_blank"><img src="https://latex.codecogs.com/gif.latex?i" title="i" /></a> belongs to an industry <a href="https://www.codecogs.com/eqnedit.php?latex=\ell" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\ell" title="\ell" /></a>, where <a href="https://www.codecogs.com/eqnedit.php?latex=N" target="_blank"><img src="https://latex.codecogs.com/gif.latex?N" title="N" /></a> the number of stocks.

Then, we construct the hierarchical model

<a href="https://www.codecogs.com/eqnedit.php?latex=\begin{align*}\phi^m_j&space;&\sim&space;\mathcal{N}(0,\&space;16\sigma_j^2)\\&space;\phi^s_{k,j}&space;&\sim&space;\mathcal{N}(\phi^m_j,\&space;4\sigma_j^2)\\&space;\phi^\iota_{\ell,j}&space;&\sim&space;\mathcal{N}(\phi^s_{\text{sec}(\ell),j},\&space;\sigma_j^2)\\&space;\phi_{i,j}&space;&\sim&space;\mathcal{N}(\phi^\iota_{\text{ind}(i),j},\&space;\tfrac{1}{4}\sigma_j^2)\\&space;\psi^m&space;&\sim&space;\mathcal{N}(0,\&space;16)\\&space;\psi_k^s&space;&\sim&space;\mathcal{N}(\psi^m,\&space;4)\\&space;\psi^\iota_{\ell}&space;&\sim&space;\mathcal{N}(\psi_{\text{sec}(\ell)}^s,\&space;1)\\&space;\psi_{i}&space;&\sim&space;\mathcal{N}(\psi^\iota_{\text{ind}(i)},\&space;\tfrac{1}{4})\\&space;y_{t,i}&space;&\sim&space;\mathcal{N}\left(\sum_{j=0}^{D}\phi_{i,j}\,\tau_t^j,&space;\&space;\log^2(1&space;&plus;&space;e^{\psi_i&space;&plus;&space;|1&space;-&space;\tau_t|})\right)&space;\end{align*}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\begin{align*}\phi^m_j&space;&\sim&space;\mathcal{N}(0,\&space;16\sigma_j^2)\\&space;\phi^s_{k,j}&space;&\sim&space;\mathcal{N}(\phi^m_j,\&space;4\sigma_j^2)\\&space;\phi^\iota_{\ell,j}&space;&\sim&space;\mathcal{N}(\phi^s_{\text{sec}(\ell),j},\&space;\sigma_j^2)\\&space;\phi_{i,j}&space;&\sim&space;\mathcal{N}(\phi^\iota_{\text{ind}(i),j},\&space;\tfrac{1}{4}\sigma_j^2)\\&space;\psi^m&space;&\sim&space;\mathcal{N}(0,\&space;16)\\&space;\psi_k^s&space;&\sim&space;\mathcal{N}(\psi^m,\&space;4)\\&space;\psi^\iota_{\ell}&space;&\sim&space;\mathcal{N}(\psi_{\text{sec}(\ell)}^s,\&space;1)\\&space;\psi_{i}&space;&\sim&space;\mathcal{N}(\psi^\iota_{\text{ind}(i)},\&space;\tfrac{1}{4})\\&space;y_{t,i}&space;&\sim&space;\mathcal{N}\left(\sum_{j=0}^{D}\phi_{i,j}\,\tau_t^j,&space;\&space;\log^2(1&space;&plus;&space;e^{\psi_i&space;&plus;&space;|1&space;-&space;\tau_t|})\right)&space;\end{align*}" title="\begin{align*}\phi^m_j &\sim \mathcal{N}(0,\ 16\sigma_j^2)\\ \phi^s_{k,j} &\sim \mathcal{N}(\phi^m_j,\ 4\sigma_j^2)\\ \phi^\iota_{\ell,j} &\sim \mathcal{N}(\phi^s_{\text{sec}(\ell),j},\ \sigma_j^2)\\ \phi_{i,j} &\sim \mathcal{N}(\phi^\iota_{\text{ind}(i),j},\ \tfrac{1}{4}\sigma_j^2)\\ \psi^m &\sim \mathcal{N}(0,\ 16)\\ \psi_k^s &\sim \mathcal{N}(\psi^m,\ 4)\\ \psi^\iota_{\ell} &\sim \mathcal{N}(\psi_{\text{sec}(\ell)}^s,\ 1)\\ \psi_{i} &\sim \mathcal{N}(\psi^\iota_{\text{ind}(i)},\ \tfrac{1}{4})\\ y_{t,i} &\sim \mathcal{N}\left(\sum_{j=0}^{D}\phi_{i,j}\,\tau_t^j, \ \log^2(1 + e^{\psi_i + |1 - \tau_t|})\right) \end{align*}" /></a>

Parameters at market-level <a href="https://www.codecogs.com/eqnedit.php?latex=\phi^m\text{&space;and&space;}\psi^m" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\phi^m\text{&space;and&space;}\psi^m" title="\phi^m\text{ and }\psi^m" /></a> are prior means for sector-level parameters <a href="https://www.codecogs.com/eqnedit.php?latex=\phi^s\text{&space;and&space;}\psi^s" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\phi^s\text{&space;and&space;}\psi^s" title="\phi^s\text{ and }\psi^s" /></a> , which in turn are prior means for industry-level parameters <a href="https://www.codecogs.com/eqnedit.php?latex=\phi^\iota\text{&space;and&space;}\psi^\iota" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\phi^\iota\text{&space;and&space;}\psi^\iota" title="\phi^\iota\text{ and }\psi^\iota" /></a> ; finally, the latter are prior means for stock-level parameters <a href="https://www.codecogs.com/eqnedit.php?latex=\phi\text{&space;and&space;}\psi." target="_blank"><img src="https://latex.codecogs.com/gif.latex?\phi\text{&space;and&space;}\psi." title="\phi\text{ and }\psi." /></a> Components of the parameters at each level are supposed to be conditionally independent given the parameters at the level above in the hierarchy. 

Whereas <a href="https://www.codecogs.com/eqnedit.php?latex=\phi" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\phi" title="\phi" /></a> are used to determine the coefficients of the polynomial model,  <a href="https://www.codecogs.com/eqnedit.php?latex=\psi" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\psi" title="\phi" /></a> are used to determine the scales of the likelihood function. The likelihood, defined in the last line of the hierarchical model, is a Gaussian centred at the polynomial model, with scales that become larger and larger the further the time index <a href="https://www.codecogs.com/eqnedit.php?latex=t" target="_blank"><img src="https://latex.codecogs.com/gif.latex?t" title="t" /></a> gets from the current time <a href="https://www.codecogs.com/eqnedit.php?latex=T" target="_blank"><img src="https://latex.codecogs.com/gif.latex?T" title="T" /></a>. In other words, recent data are weighted more than older ones, which get less and less importance the older they get.

In order to estimate parameters, we condition on log-prices <a href="https://www.codecogs.com/eqnedit.php?latex=y_{t,i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?y_{t,i}" title="y_{t,i}" /></a>, for all <a href="https://www.codecogs.com/eqnedit.php?latex=t=1,\dots&space;T" target="_blank"><img src="https://latex.codecogs.com/gif.latex?t=1,\dots&space;T" title="t=1,\dots T" /></a>, then we estimate the mode of the posterior distribution, also known as Maximum-A-Posteriori (MAP). From a frequentist statistics perspective, this corresponds to a polynomial regression task where we minimise a regularised and weighted mean-squared error loss. A plot showing the loss decay during training can be saved in the current directory as `loss_decay.png` by adding the flag `--plot-loss` in the command line.

Obtained our estimates <a href="https://www.codecogs.com/eqnedit.php?latex=\hat\phi^m,\hat\phi^s,\hat\phi^\iota,\hat\phi,\hat\psi^m,\hat\psi^s,\hat\psi^\iota\text{&space;and&space;}&space;\hat\psi" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat\phi^m,\hat\phi^s,\hat\phi^\iota,\hat\phi,\hat\psi^m,\hat\psi^s,\hat\psi^\iota\text{&space;and&space;}&space;\hat\psi" title="\hat\phi^m,\hat\phi^s,\hat\phi^\iota,\hat\phi,\hat\psi^m,\hat\psi^s,\hat\psi^\iota\text{ and } \hat\psi" /></a>, we can use the likelihood mean <a href="https://www.codecogs.com/eqnedit.php?latex=\hat&space;y_{t,i}=\sum_{j=0}^{D}\hat\phi_{i,j}\,\tau_t^j" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat&space;y_{t,i}=\sum_{j=0}^{D}\hat\phi_{i,j}\,\tau_t^j" title="\hat y_{t,i}=\sum_{j=0}^{D}\hat\phi_{i,j}\,\tau_t^j" /></a> as an estimator of the log-prices for any time in the past, as well as a predictor for times in the short future. As a measure of uncertainty, we take the learned scale of the likelihood, that is <a href="https://www.codecogs.com/eqnedit.php?latex=\hat\sigma_{t,i}=\log(1&plus;e^{\hat\psi_i&plus;|1-\tau_t|})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat\sigma_{t,i}=\log(1&plus;e^{\hat\psi_i&plus;|1-\tau_t|})" title="\hat\sigma_{t,i}=\log(1+e^{\hat\psi_i+|1-\tau_t|})" /></a>.

We use the estimates above to select the order <a href="https://www.codecogs.com/eqnedit.php?latex=D" target="_blank"><img src="https://latex.codecogs.com/gif.latex?D" title="D" /></a> of the polynomial. For each candidate order, we train the model with data up to 5 trading days before the current date, then predict the last 5 trading days and test against actual observations. If the likelihood model fits well the data, we should have that the empirical second moment  <a href="https://www.codecogs.com/eqnedit.php?latex=\frac{1}{5N}\sum_{r=0}^4\sum_{i=1}^N\left(\frac{\hat&space;y_{T-r,i}-y_{T-r,i}}{\hat\sigma_{T-r,i}}\right)^2" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\frac{1}{5N}\sum_{r=0}^4\sum_{i=1}^N\left(\frac{\hat&space;y_{T-r,i}-y_{T-r,i}}{\hat\sigma_{T-r,i}}\right)^2" title="\frac{1}{5N}\sum_{r=0}^4\sum_{i=1}^N\left(\frac{\hat y_{T-r,i}-y_{T-r,i}}{\hat\sigma_{T-r,i}}\right)^2" /></a> is approximately 1, where <a href="https://www.codecogs.com/eqnedit.php?latex=\hat&space;y_{T-r,i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat&space;y_{T-r,i}" title="\hat y_{T-r,i}" /></a> and <a href="https://www.codecogs.com/eqnedit.php?latex=\hat\sigma_{T-r,i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat\sigma_{T-r,i}" title="\hat\sigma_{T-r,i}" /></a> are the estimators described above, while <a href="https://www.codecogs.com/eqnedit.php?latex=y_{T-r,i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?y_{T-r,i}" title="y_{T-r,i}" /></a> are actual log-price observations. Thus, we compute their absolute distance and select the polynomial order that makes it the smallest.

Given the selected model complexity, Volatile trains the model and provides a rating for each stock by introducing the following score:

<a href="https://www.codecogs.com/eqnedit.php?latex=\text{score}_i=\frac{\hat&space;y_{T&plus;5,i}-y_{T,i}}{\hat\sigma_{T&plus;5,i}}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\text{score}_i=\frac{\hat&space;y_{T&plus;5,i}-y_{T,i}}{\hat\sigma_{T&plus;5,i}}" title="\text{score}_i=\frac{\hat y_{T+5,i}-y_{T,i}}{\hat\sigma_{T+5,i}}" /></a>

where <a href="https://www.codecogs.com/eqnedit.php?latex=\inline&space;y_{T,i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\inline&space;y_{T,i}" title="y_{T,i}" /></a> is the last available log-price, <a href="https://www.codecogs.com/eqnedit.php?latex=\inline&space;\hat&space;y_{T&plus;5,i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\inline&space;\hat&space;y_{T&plus;5,i}" title="\hat y_{T+5,i}" /></a> is its prediction in 5 trading days (usually, that corresponds to the log-price in one week) and <a href="https://www.codecogs.com/eqnedit.php?latex=\inline&space;\hat\sigma_{T&plus;5,i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\inline&space;\hat\sigma_{T&plus;5,i}" title="\hat\sigma_{T+5,i}" /></a> is the estimated standard deviation of the prediction. If the future prediction is larger than the current price, the score will be positive; the larger the difference and the more confident we are about the prediction (or equivalently, the smaller the standard deviation is), the more positive will be the score. We can reason similarly if the score is negative. In other words, a large positive score indicates that the current price is undervalued with respect to its stock trend, therefore an opportunity to buy; a large negative score indicates, vice versa, that the current price is overvalued with respect to its stock trend, therefore a moment to sell. 

Then, stocks are rated according to the following criteria:
- HIGHLY BELOW TREND if <a href="https://www.codecogs.com/eqnedit.php?latex=\text{score}_i>3" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\text{score}_i>3" title="\text{score}_i>3" /></a>; 
- BELOW TREND if <a href="https://www.codecogs.com/eqnedit.php?latex=2<\text{score}_i<=3" target="_blank"><img src="https://latex.codecogs.com/gif.latex?2<\text{score}_i<=3" title="2<\text{score}_i<=3" /></a>;
- ALONG TREND if <a href="https://www.codecogs.com/eqnedit.php?latex=-2<\text{score}_i<=2" target="_blank"><img src="https://latex.codecogs.com/gif.latex?-2<\text{score}_i<=2" title="-2<\text{score}_i<=2" /></a>;
- ABOVE TREND if <a href="https://www.codecogs.com/eqnedit.php?latex=-3<\text{score}_i<=-2" target="_blank"><img src="https://latex.codecogs.com/gif.latex?-3<\text{score}_i<=-2" title="-3<\text{score}_i<=-2" /></a>;
- HIGHLY ABOVE TREND if <a href="https://www.codecogs.com/eqnedit.php?latex=\text{score}_i<=-3" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\text{score}_i<=-3" title="\text{score}_i<=-3" /></a>.

Because we model log-prices as a Gaussian, the distribution of prices is a log-Normal distribution, whose mean and standard deviation can be derived in closed form from the estimators <a href="https://www.codecogs.com/eqnedit.php?latex=\hat&space;y_{t,i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat&space;y_{t,i}" title="\hat y_{t,i}" /></a> and <a href="https://www.codecogs.com/eqnedit.php?latex=\hat\sigma_{t,i}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat\sigma_{t,i}" title="\hat\sigma_{t,i}" /></a>. We use log-Normal distribution statistics at times <a href="https://www.codecogs.com/eqnedit.php?latex=t=1\dots,T" target="_blank"><img src="https://latex.codecogs.com/gif.latex?t=1\dots,T" title="t=1\dots,T" /></a> to produce the stock estimation plot and at time <a href="https://www.codecogs.com/eqnedit.php?latex=T&plus;1" target="_blank"><img src="https://latex.codecogs.com/gif.latex?T&plus;1" title="T+1" /></a> to fill the prediction table. In order to produce the market, sector and industry estimation plots, we proceed analogously but with estimators at respective levels, that is <a href="https://www.codecogs.com/eqnedit.php?latex=\hat&space;y^m_{t}=\sum_{j=0}^{D}\hat\phi^m_{j}\,\tau_t^j" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat&space;y^m_{t}=\sum_{j=0}^{D}\hat\phi^m_{j}\,\tau_t^j" title="\hat y^m_{t}=\sum_{j=0}^{D}\hat\phi^m_{j}\,\tau_t^j" /></a> and <a href="https://www.codecogs.com/eqnedit.php?latex=\hat\sigma^m_{t}=\log(1&plus;e^{\hat\psi^m&plus;|1-\tau_t|})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat\sigma^m_{t}=\log(1&plus;e^{\hat\psi^m&plus;|1-\tau_t|})" title="\hat\sigma^m_{t}=\log(1+e^{\hat\psi^m+|1-\tau_t|})" /></a> for market, <a href="https://www.codecogs.com/eqnedit.php?latex=y^s_{t,k}=\sum_{j=0}^{D}\hat\phi^s_{k,j}\,\tau_t^j" target="_blank"><img src="https://latex.codecogs.com/gif.latex?y^s_{t,k}=\sum_{j=0}^{D}\hat\phi^s_{k,j}\,\tau_t^j" title="y^s_{t,k}=\sum_{j=0}^{D}\hat\phi^s_{k,j}\,\tau_t^j" /></a> and <a href="https://www.codecogs.com/eqnedit.php?latex=\hat\sigma^s_{t,k}=\log(1&plus;e^{\hat\psi^s_k&plus;|1-\tau_t|})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat\sigma^s_{t,k}=\log(1&plus;e^{\hat\psi^s_k&plus;|1-\tau_t|})" title="\hat\sigma^s_{t,k}=\log(1+e^{\hat\psi^s_k+|1-\tau_t|})" /></a> for sector, <a href="https://www.codecogs.com/eqnedit.php?latex=\hat&space;y^\iota_{t,\ell}=\sum_{j=0}^{D}\hat\phi^\iota_{\ell,j}\,\tau_t^j" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat&space;y^\iota_{t,\ell}=\sum_{j=0}^{D}\hat\phi^\iota_{\ell,j}\,\tau_t^j" title="\hat y^\iota_{t,\ell}=\sum_{j=0}^{D}\hat\phi^\iota_{\ell,j}\,\tau_t^j" /></a> and <a href="https://www.codecogs.com/eqnedit.php?latex=\hat\sigma^\iota_{t,\ell}=\log(1&plus;e^{\hat\psi^\iota_\ell&plus;|1-\tau_t|})" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\hat\sigma^\iota_{t,\ell}=\log(1&plus;e^{\hat\psi^\iota_\ell&plus;|1-\tau_t|})" title="\hat\sigma^\iota_{t,\ell}=\log(1+e^{\hat\psi^\iota_\ell+|1-\tau_t|})" /></a> for industry.
