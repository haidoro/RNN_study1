# Word2Vecにチャレンジ

`word2vec_neko.ipynb`ファイル

### Word2Vecとは
ニューラルネットワークを使って単語を処理する場合、そのまま処理することができません。
そのため、単語を固定長ベクトルに変換する必要があります。

Word2Vecは単語のベクトル表現をするためのものです。

例  
You say goodby and i say hello.

この文章の単語にそれぞれidをつけていきます。

you : 10   
say : 88  
goodby : 5  
and : 91    
i : 22    
say : 7    
hello. :35  
などのように一意のidを振ります。そうすることで`{10,88,5,91,22,7,35}`ようなベクトルにすることができます。
こうすることで、似た特徴を持つ単語を推測することができるようになります。
そして、これらを応用することで翻訳や自動応答システムなどに活用されています。

* この辺の仕組みは別途解説予定

## 青空文庫の解析でword2vecを使って見る

### gensimの導入

#### Anacondaナビゲータからの導入方法

前回作成した仮想　morphに切り替えてメニューから「Enviroments」とします。 
 
右側にはinstallされたライブラリ等の一覧が表示されています。
上にある「Installed」のプルダウンメニューから「Not Installed」を選択します。そしてすぐ近くにある「Search Packages」に「gensim」と入力すると「gensim」が表示されます。

あとはチェックボックスにチェックを入れるとインストールしてくれます。

### Word2Vecの実装

コードは`word2vec_neko.ipynb`ファイルを参照ください。

`import re`のreとはRegular Expressionのことで「正規表現」を使用するために必要なモジュールです。

***今回の青空文庫の「我輩は猫である」はzipファイルから一旦手動で展開しておきます。***

そのため、ファイルの読み込みは次のようにしています。

```
binarydata = open("wagahaiwa_nekodearu.txt",'rb').read()
```

ただしこのデータはバイナリデータだから、一旦デコードします。
つまりデータ型から文字列に変換します。

```
text = binarydata.decode('shift_jis')
```

次に、ヘッダー部分とフッター部分の記述は余計ですから`split()`関数で削除します。  
この辺の処理は普通にPythonと正規表現の処理ですから臨機応変に対処します。

今回使用しているのは、正規表現の`re.split()`関数を使っています。 
`split()`関数は引数に指定した内容を区切りとして文字列をリスト化して返すものです。   
従って、`{5,}`はハイフンが5以上繰り返された場合の正規表現。が区切りとなるわけですから、ヘッダー上部と下部のハイフンの繰り返しが区切りとなって分割されます。  
そして分割された内容がリスト化されるわけです。
ヘッダー上部にある表題と著者名が0番目、ヘッダー部分が1番目、そしてそのあとの本文が2番目となるわけです。
このようにして本文を切り取っています。  

```
text = re.split(r'\-{5,}',text)[2]
```
フッター部分もヘッダー同様の考え方です。フッターの先頭にある「底本：」（コロンは全角）を目印に分割します。それより上が本文で0番目ですから本文部分を分割して変数textに代入します。

```
text = re.split(r'底本：',text)[0]
```

`strip()`関数は文字列の両端から指定文字をとるものです。
引数を指定しない場合は、「空白文字」を削除します。

### 次に形態素解析を行う

```
t =Tokenizer()
lines = text.split("\r\n")
```

1. ルビや入力注を取ります。  
2. その後、形態素に分解します。  
3. 品詞の前の先頭を取得する
その後わかち書き状態にする。

```
for line in lines:
    s = line
    s = s.replace('|','')
    s = re.sub(r'《.+?》','',s) # ルビをとる
    s = re.sub(r'［＃.+?］','',s) # 入力注をとる
    tokens = t.tokenize(s)
    r = []
    for token in tokens:
        if token.base_form == "*":
            w = token.surface
        else:
            w = token.base_form
        ps = token.part_of_speech
        hinshi = ps.split(',')[0]
        if hinshi in ['名詞','形容詞','動詞','記号']:
            r.append(w)
    rl = (" ".join(r)).strip()
    results.append(rl)
    print(rl)
```

## Word2Vecで解析する
`word2vec.LineSentence`関数を使う
モデルを生成して保存します。

```
data = word2vec.LineSentence(wakachigaki_file)
model = word2vec.Word2Vec(data,size=200,window=10,hs=1,min_count=2,sg=1)
model.save('neko.model')
print('Completed')
```

モデルの確認
猫に近い言葉は何か表示されます。

```
model.most_similar(positive=['猫'])
```

結果
('餅屋', 0.7562851309776306),
('高慢', 0.7441864609718323),
('己', 0.7381236553192139),
('冒険', 0.7341221570968628),
('軽侮', 0.7305941581726074),
('自白', 0.7288652062416077),
('義侠', 0.7287881970405579),
('種族', 0.7248501777648926),
('軽蔑', 0.7229416966438293),
('精密', 0.713079571723938)

ん〜微妙。。。
