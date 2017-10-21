# Keras使用陷阱

這裡歸納了Keras使用過程中的一些常見陷阱和解決方法，如果你的模型怎麼調都搞不對，或許你有必要看看是不是掉進了哪個獵人的陷阱，成為了一隻嗷嗷待宰（？ ）的獵物

Keras陷阱不多，我們保持更新，希望能做一個陷阱大全

內有惡犬，小心喲

## TF卷積核與TH卷積核

Keras提供了兩套後端，Theano和Tensorflow，這是一件幸福的事，就像手中拿著饅頭，想蘸紅糖蘸紅糖，想蘸白糖蘸白糖

如果你從無到有搭建自己的一套網絡，則大可放心。但如果你想使用一個已有網絡，或把一個用th/tf
訓練的網絡以另一種後端應用，在載入的時候你就應該特別小心了。

卷積核與所使用的後端不匹配，不會報任何錯誤，因為它們的shape是完全一致的，沒有方法能夠檢測出這種錯誤。

在使用預訓練模型時，一個建議是首先找一些測試樣本，看看模型的表現是否與預計的一致。

如需對卷積核進行轉換，可以使用utils.convert_all_kernels_in_model對模型的所有捲積核進行轉換

## 向BN層中載入權重
如果你不知道從哪里淘來一個預訓練好的BN層，想把它的權重載入到Keras中，要小心參數的載入順序。

一個典型的例子是，將caffe的BN層參數載入Keras中，caffe的BN由兩部分構成，bn層的參數是mean，std，scale層的參數是gamma，beta

按照BN的文章順序，似乎載入Keras BN層的參數應該是[mean, std, gamma, beta]

然而不是的，Keras的BN層參數順序應該是[gamma, beta, mean, std]，這是因為gamma和beta是可訓練的參數，而mean和std不是

Keras的可訓練參數在前，不可訓練參數在後

錯誤的權重順序不會引起任何報錯，因為它們的shape完全相同

## shuffle和validation_split的順序

模型的fit函數有兩個參數，shuffle用於將數據打亂，validation_split用於在沒有提供驗證集的時候，按一定比例從訓練集中取出一部分作為驗證集

這裡有個陷阱是，程序是先執行validation_split，再執行shuffle的，所以會出現這種情況：

假如你的訓練集是有序的，比方說正樣本在前負樣本在後，又設置了validation_split，那麼你的驗證集中很可能將全部是負樣本

同樣的，這個東西不會有任何錯誤報出來，因為Keras不可能知道你的數據有沒有經過shuffle，保險起見如果你的數據是沒shuffle過的，最好手動shuffle一下

## Merge層的層對象與函數方法

Keras定義了一套用於融合張量的方法，位於keras.layers.Merge，裡面有兩套工具，以大寫字母開頭的是Keras Layer類，使用這種工具是需要實例化一個Layer對象，然後再使用。以小寫字母開頭的是張量函數方法，本質上是對Merge Layer對象的一個包裝，但使用更加方便一些。注意辨析。


## 未完待續

如果你在使用Keras中遇到難以察覺的陷阱，請發信到moyan_work@foxmail.com說明~贈人玫瑰，手有餘香，前人踩坑，後人沾光，有道是我不入地獄誰入地獄，願各位Keras使用者積極貢獻Keras陷阱。老規矩，陷阱貢獻者將被列入致謝一欄