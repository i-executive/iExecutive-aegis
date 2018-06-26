# テスト用の環境について

# etherium のテスト用ネットワークを構築

VPNで接続して利用する。

# ethereum の状態のモニタ

http://10.16.16.4:53000

# ネットワークへの接続

gethのコンソールで、以下を入力

> admin.addPeer("enode://f465c9773019eddb28c695030b23f0cbb5020be836652c85712e839a1bfa9945c4fa5da121d541ffbec65e0e72164fa529f4d7ef7c448542372b52e89deef7da@10.16.16.4:60303");

# Walletの設定

rpc http://[node-ip]:8545

ether wallet (mac)
> open -a "/Applications/Ethereum Wallet.app/contents/macos/Ethereum Wallet" --args --rpc http://localhost:8545



