---
layout: post
title:  "Starting on smart contract with Web3j"
date: 2018-07-13
tags:
- ethereum
- smartcontract
- java
permalink: 
description: starting-smart-contract
---


It's been a few year that blockchain became one of hottest trend in tech/economic field, and related skills skyrocketed, something like AI, Cloud skills. But not only that reason, it has quite facinating features for engineer, and one of them are `smart contract`.
(I'm not gonna describe ethereum, blockchain in detail here. I'm also still studying on it. You can find lot of references in google...)


## Smart contract?
In document, it defined as 'account holding objects on the ethereum blockchain', but it has more complex meaning than 'account'. Describing more program-like, it can be 'group of function uploaded on ethereum blockchain'. They contain code functions and can interact with other contracts, make decisions, store data, and send ether to others. 
Maybe we can think similarly with API server, because it also offers information by functions. Difference is that execution of smart contract helds on ethereum network, and more important, they cannot be terminated after upload, until network destroyed(blockchain is decentralized system, as you know).


## Solidity
Well, because it was quite a new concept for me, I couldn't get it with this explanation. 

![Screenshot](/assets/post_img/starting-smart-contract/ethereum.png)

There are several blockchain platform, but most of will recommend `Ethereum` to startover. It is one of the oldest, and still the most biggest/popular platform supporting smart contract, and has lot of references and modules. You can find some of related project [here](https://github.com/ethereum)(there are more than this!).

To implement smart contract functions, you need to use language which Ethreum supports. There are several languages, but `Solidity` is most popular for now.


{% highlight shell %}
{% raw %}

{% endraw %}
{% endhighlight %}

{% highlight shell %}
{% raw %}
pragma solidity ^0.4.0;

contract SimpleGetSet {
    uint storedData;

    function set(uint x) public {
        storedData = x;
    }

    function get() public constant returns (uint) {
        return storedData;
    }
}
{% endraw %}
{% endhighlight %}
This is the basic sample in solidity guide document. As you can see, if you are use to in programming language such as JS, Java, you could assume what this function does. Contract name is `SimpleGetSet`, and it has two methods...`set(uint)` receives unsigned integer value, and `get()` sends received value.


## Ethereum network
If you want to create and use private network in your computer for development, you could simply do it by installing [geth](https://github.com/ethereum/go-ethereum) or [parity](https://www.parity.io/). But if you want more simple, you could use [infura](https://infura.io/) platform, which offers free network. In my case, running in localhost takes a lot of CPU/memory, so went on to this.

![Screenshot](/assets/post_img/starting-smart-contract/infura.png)

If you sign on and create a project, you will get API key for certify, and can select endpoint to access. 



## Web3...whatever
You can access to Ethereum network with API, and there are well-prepared wrapper for most of the programming languages, usually named as `Web3...`. Most popular one is Javascript wrapper [Web3](https://github.com/ethereum/web3.js), and there are for [Python(Web3py)](https://github.com/ethereum/web3.py), [Java(Web3j)](https://github.com/web3j/web3j), and else. I will work on with Java here(no reason for this...).

{% highlight java %}
{% raw %}
public static void main(String[] args) {

    String infuraKey = "your-key";
    String serviceUrl = "https://rinkeby.infura.io/" + infuraKey;
    Web3j web3j = Web3j.build(new HttpService(serviceUrl));
    try {
        Web3ClientVersion web3ClientVersion = web3j.web3ClientVersion().send();
        System.out.println("Web3 Client: " + web3ClientVersion.getWeb3ClientVersion());

    } catch (IOException ioe) {
        System.out.println("IOException:" + ioe);
    }
}
{% endraw %}
{% endhighlight %}

This is the basic code, only accessing to ethereum network and get ethereum network version. Value `infuraKey` and `serviceUrl` is the value which I get/selected from `infura`. Change to localhost if you are using local network. If this code goes right, you could see something like

```
Web3 Client: Geth/v1.8.3-stable/linux-amd64/go1.10
```


## 



## Reference
* https://www.ethereum.org/
* https://docs.web3j.io/
* http://solidity.readthedocs.io/

