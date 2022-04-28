### 在Event.js


```js

const { ApiPromise } = require('@polkadot/api');

const testKeyring = require('@polkadot/keyring/testing');

const { randomAsU8a } = require('@polkadot/util-crypto');

const ALICE = '5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY';
const AMOUNT = 10000;

async function main () {
    const api = await ApiPromise.create();


    const keyring = testKeyring.default();

    const { nonce } = await api.query.system.account(ALICE);

    const alicePair = keyring.getPair(ALICE);

    const recipient = keyring.addFromSeed(randomAsU8a(32)).address;

    console.log('发送', AMOUNT, '从', alicePair.address, '到', recipient, 'nonce为', nonce.toString());

    api.tx.balances
        .transfer(recipient, AMOUNT)
        .signAndSend(alicePair, { nonce }, ({ events = [], status }) => {
            console.log('交易状态:', status.type);

            if (status.isInBlock) {
                console.log('哈希', status.asInBlock.toHex());
                console.log('事件:');

                events.forEach(({ event: { data, method, section }, phase }) => {
                    console.log('\t', phase.toString(), `: ${section}.${method}`, data.toString());
                });
            } else if (status.isFinalized) {
                console.log('最终哈希', status.asFinalized.toHex());

                process.exit(0);
            }
        });
}

main().catch(console.error);
```

