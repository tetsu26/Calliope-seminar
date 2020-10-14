# DAY 3

それでは、オリジナルのエネルギーシステムモデルを作製し、分析してみよう。

## 作製するモデルの詳細

今回作成するエネルギーシステムは、対象地域を東北電力管内（東北六県+新潟県）として、電力需要のみを考慮する。各県の電力需要と、太陽光発電ポテンシャル、洋上風力発電ポテンシャルの推計値を入力とし、東北電力管内で電力を賄った際の電力の流れを可視化する。各県を以下の図のように接続し、電力を融通する。

![モデル図](images/model.svg)

## 下準備

今回はチュートリアルモデルをベースにモデルを作製する。前回と同様に、適当なディレクトリにチュートリアルモデルをコピーする。
```bash
calliope new test_model
code test_model
```
!!! Tips
    `calliope`コマンドでエラーが出る際は、`conda activate calliope`でcalliope環境に入ろう。

次にモデルで入力として使う、時系列データをダウンロードする。

[[学内限定]ダウンロードリンク](https://drive.google.com/drive/folders/1l1DxK1kyN4VkEtbbML33_aez7rf5Yxu5?usp=sharing)

## `model.yaml`を編集
`model.yaml`では、Calliopeの実行に関するすべてを設定する。以下三つに大きく分けて設定する。

:information_source: [Calliope公式ドキュメント](https://calliope.readthedocs.io/en/stable/user/config_defaults.html#config-reference-model)

### `import`

```yaml
import:  
    - 'model_config/techs.yaml' 
    - 'model_config/locations.yaml' 
    - 'scenarios.yaml' 
```
`import`にはCalliopeの解析に必要な設定ファイルを分散して書いた際に、そのファイルの場所を記述する。

各需要家や供給元のデータ、それぞれの位置情報などをすべて同じファイルに記載することもできるが、そうすると大変読みにくくなってしまうので、通常はファイルを分けて記載する。この資料でも、`techs.yaml`、`locations.yaml`に分割して記載する。

今回は同じ場所にファイルを置くので編集する必要はない。

### `model`

```yaml
model:
    name: National-scale example model
    calliope_version: 0.6.5
    timeseries_data_path: 'timeseries_data'
    subset_time: ['2005-01-01', '2005-01-05']
```
`model`には実行したいモデルの設定を記述する。

`timeseries_data_path`にはその名の通り、計算に使う時系列データの場所を指定する。

`subset_time`で使用するデータの範囲を選択することができる。実際には１年間分のデータがあったとしても、設定ファイルが正しいか確かめるために1年間の計算をいちいちする必要は無いので、一部のデータだけで確かめるためなどに使う。

今回は、例と同じ時間範囲のデータを扱うので編集する必要はない。nameは変えたい人は変えてもよい。

### `run`

```yaml
run:
    solver: cbc
    ensure_feasibility: True
    bigM: 1e6
    zero_threshold: 1e-10 
    mode: plan
    objective_options.cost_class: {monetary: 1}
```
`run`では、計算方法に関する設定を記述する。

`solver`では、最適化計算に用いるソルバーを指定する。デフォルトでは`cbc`になっているが、今回は`gurobi`を指定する。

`ensure_freasibility`は、Trueにすると、供給が需要を満たせないときでも計算を続行する。Falseにすると、需要と供給のアンマッチが発生した場合、最適化計算は失敗する。

`bigM`は、需要を満たせないときに使用される。モデルが到達しうる最大のコストくらいの大きさの値であるべきであり、大きすぎると計算が収束しない。

`zero_threshold`よりも小さい値はおそらく浮動小数点エラーであるため、削除する。

`mode`では最適化計算のモードを選択する。Calliopeには`plan`、`operate`、`spores`の三つのモードがある。planモードでは容量がモデルによって決定される、つまり、ゼロからエネルギーシステムを設計する際に適している。operateモードでは、容量は固定され、システムがモデル予測制御（MPC）によって運用される。つまり、既存の設備を用いたエネルギーシステムの最適化に適している。sporesモードでは、最初にplanモードで実行し、その後、似たようなコストだが、設備の容量と場所がもっとも異なる物を見つける。そのため、モデルはN回実行される。

今回はsolverをgurobiに変更するのみで良い。

:information_source: [MPCに関して深く知りたい方はこちら](https://myenigma.hatenablog.com/entry/2016/07/25/214014)
:information_source: [MPCとエネルギーシステム](https://orbit.dtu.dk/en/publications/model-predictive-control-for-smart-energy-systems-2)

## `techs.yaml`を編集

`techs.yaml`は`model_config`フォルダの中にある。

### 技術の種類

Calliopeでは、技術をあらかじめ以下の7種に分類し、簡単に扱えるよう工夫されている。

| 技術の種類 | 詳細 |
| -------- | ---- |
| `supply` | キャリアにエネルギーを供給し、正の資源を持つ。 |
| `supply_plus` | キャリアにエネルギーを供給し、正の資源を持つ。`supply`と違い、効率や貯蔵などの制約条件を掛けられる。 |
| `demand` | キャリアからエネルギーを受け取り、消費する。負の資源を持つ。 |
| `storage` | エネルギーを貯蔵する。 |
| `transmission` | エネルギーをある場所から違う場所へと輸送する。 |
| `conversion` | エネルギーをあるキャリアから違うキャリアへと変換する。 |
| `conversion_plus` | エネルギーをあるキャリア（複数）から違うキャリア（複数）へと変換する。 |

### 制約条件

Calliopeでは、各技術ノードに様々な制約条件を足すことができる。以下でよく使いそうな制約条件をいくつか説明するが、詳しくは[公式ドキュメント](https://calliope.readthedocs.io/en/stable/user/config_defaults.html)を参考にしてほしい。

#### 技術に関して
| よく使いそうな制約条件 | 単位 | 説明 |
| ------- | -------- | ------ |
| `resource` | kWh,kWh/m2,kWh/kW | 使えるリソースの最大量 単位は`resource_unit`で指定可能 |
| `energy_eff` | 割合 | 発電効率 |
| `energy_cap_max` | kW | 導入可能な設備容量の最大値 |
| `energy_cap_max_systemwide` | kW | システム全体での導入可能な設備容量の最大値の合計 |
| `energy_ramping` | 割合/hour | ランピング制約を指定し、電力の生産量の変化率を設定 |
| `energy_cap_per_unit` | kW/unit | 一つの装置当たりの設備容量 |
| `lifetime` | year | 設備の寿命 |

#### コストに関して
| よく使いそうな制約条件 | 単位 | 説明 |
| ------- | -------- | ------ |
| `energy_cap` | /kW_gross | 容量あたりの価格 |
| `interest_rate` | - | 割引率 |
| `om_annual` | /kW_energy_cap | 年間のOperation & Maintenance費 |
| `om_con` | /kWh | キャリアのconsumption代、すなわち燃料費 |
| `om_prod` | /kWh | キャリアをproductionするのに要する費用、再エネ発電などで用いられる |

!!! Note "割引率とは"
    > 割引率とは、将来の価値を現在の価値に直すために用いる率のことをいいます。 利回りを考慮すれば現在の通貨の価値と将来の通貨の価値とでは価値が違うために、将来の通貨の価値を現在の通貨の価値に換算するために用いる率のことを指します。 :information_source: [出典](https://www.shinnihon.or.jp/corporate-accounting/glossary/retirement-benefits/waribiki-ritu.html)

    つまり、お金の価値は時代によって変わる（例えば今の10円と明治時代の10円では価値が全く違う）ので、それを補正するために割引率が設定される。

### 具体例

```yaml
techs:
    ccgt:
        essentials:
            name: 'Combined cycle gas turbine'
            color: '#E37A72'
            parent: supply
            carrier_out: power
        constraints:
            resource: inf
            energy_eff: 0.5
            energy_cap_max: 40000  # kW
            energy_cap_max_systemwide: 100000  # kW
            energy_ramping: 0.8
            lifetime: 25
        costs:
            monetary:
                interest_rate: 0.10
                energy_cap: 750  # USD per kW
                om_con: 0.02  # USD per kWh
```
2行目の`ccgt:`は、任意に名づけることができる。5行目の`color:`はCalliopeで作図する際に使用される色である。`carrier_out:`で出力されるエネルギーのキャリアを選ぶことができる。`parent:`でこの技術ノードに当てはまる技術の種類を指定する。

### `tech_groups`を作製
`tech_groups`を作ることで、設定を簡略化できる。例えば、
```yaml
tech_groups:
    supply_power_plus:
        essentials:
            parent: supply_plus
            carrier: electricity
```
のようにすると、`essentials.parent`に`supply_power_plus`を指定することで、`carrier`が`electricity`に指定される。複数の要素に共通する部分をまとめておくと、少し楽である。

### 今回使う`techs:`など
今回使う`techs:`などを下に折りたたんで貼る。

<details>

<summary>コード</summary>

```yaml
tech_groups:
    supply_power_plus:
        essentials:
            parent: supply_plus
            carrier: electricity

techs:
    ccgt:
        essentials:
            name: 'Combined cycle gas turbine'
            color: '#E37A72'
            parent: supply
            carrier_out: electricity
        constraints:
            resource: inf
            energy_eff: 0.5
            energy_cap_max: 1400000  # kW
            energy_cap_max_systemwide: 10000000  # kW
            energy_ramping: 0.8
            lifetime: 25
        costs:
            monetary:
                interest_rate: 0.02
                energy_cap: 120000  # JPY per kW
                om_con: 0.45  # JPY per kWh
                om_annual: 20000 # JPY per kW
    pv:
        essentials:
            name: 'Solar photovoltaic power'
            color: '#F9CF22'
            parent: supply_power_plus
        constraints:
            resource_unit: energy
            energy_eff: 1
            lifetime: 25
        costs:
            monetary:
                interest_rate: 0.02
                energy_cap: 294000 # JPY per kW
                om_annual: 6000 # JPY per kW
    offwind:
        essentials:
            name: 'Offshore wind power'
            color: '#326da8'
            parent: supply_power_plus
        constraints:
            resource_unit: energy
            energy_eff: 1
            lifetime: 25
        costs:
            monetary:
                interest_rate: 0.02
                energy_cap: 565000 # JPY per kW
                om_annual: 22500 # JPY per kW
    demand_power:
        essentials:
            name: 'Electrical demand'
            color: '#072486'
            parent: demand
            carrier: electricity
    ac_transmission:
        essentials:
            name: 'AC power transmission'
            color: '#8465A9'
            parent: transmission
            carrier: electricity
        constraints:
            energy_eff: 0.95
            lifetime: 25
        costs:
            monetary:
                interest_rate: 0.02
```

</details>

## `locations.yaml`を編集 
`locations:`では、各ノードの物理的な位置や、導入する技術の種類をノードごとに指定することができる。`links:`では、各ノード同士の接続方法や、送電容量などを指定することができる。

### `locations`
`locations:`で前のセクションで作製した技術ノードを指定し、設定を書き換えることができる。うまく説明できないので、今回使う`locations:`を折りたたんで貼る。
<details>

<summary>コード</summary>

```yaml
locations:
    akita:
        coordinates: {lat: 39.7186, lon: 140.102334}
        techs:
            demand_power:
                constraints:
                    resource: file=demand.csv:akita
            pv:
                constraints:
                    resource: file=pv_potential.csv:akita
            offwind:
                constraints:
                    resource: file=offwind_potential.csv:akita
    aomori:
        coordinates: {lat: 40.824912, lon: 140.739977}
        techs:
            demand_power:
                constraints:
                    resource: file=demand.csv:aomori
            pv:
                constraints:
                    resource: file=pv_potential.csv:aomori
            offwind:
                constraints:
                    resource: file=offwind_potential.csv:aomori
    fukushima:
        coordinates: {lat: 37.750109, lon: 140.466692}
        techs:
            demand_power:
                constraints:
                    resource: file=demand.csv:fukushima
            pv:
                constraints:
                    resource: file=pv_potential.csv:fukushima
            offwind:
                constraints:
                    resource: file=offwind_potential.csv:fukushima
    iwate:
        coordinates: {lat: 39.703469, lon: 141.152589}
        techs:
            demand_power:
                constraints:
                    resource: file=demand.csv:iwate
            pv:
                constraints:
                    resource: file=pv_potential.csv:iwate
            offwind:
                constraints:
                    resource: file=offwind_potential.csv:iwate
            ccgt:
    miyagi:
        coordinates: {lat: 38.268672, lon: 140.87217}
        techs:
            demand_power:
                constraints:
                    resource: file=demand.csv:miyagi
            pv:
                constraints:
                    resource: file=pv_potential.csv:miyagi
            offwind:
                constraints:
                    resource: file=offwind_potential.csv:miyagi
            ccgt:
    yamagata:
        coordinates: {lat: 38.240409, lon: 140.363541}
        techs:
            demand_power:
                constraints:
                    resource: file=demand.csv:yamagata
            pv:
                constraints:
                    resource: file=pv_potential.csv:yamagata
            offwind:
                constraints:
                    resource: file=offwind_potential.csv:yamagata
            ccgt:
    niigata:
        coordinates: {lat: 37.901983, lon: 139.023298}
        techs:
            demand_power:
                constraints:
                    resource: file=demand.csv:niigata
            pv:
                constraints:
                    resource: file=pv_potential.csv:niigata
            offwind:
                constraints:
                    resource: file=offwind_potential.csv:niigata
```
</details>

このように、`techs:`以下に記述することで上書きできる。今回は、各県の需要量やポテンシャルを`resource:`に指定している。ちなみに資源をcsvから引用する際は、`resource: file=offwind_potential.csv:niigata`と書き、`:niigata`では列を指定している。

### `links`
`links`では`locations`で指定した要素同士の関係について指定する。これも今回使うものを下に折りたたんで貼る。

<details>

<summary>コード</summary>

```yaml
links:
    akita,aomori:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 1340000 # kWh
    aomori,iwate:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 17300000
    akita,iwate:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 660000
    akita,miyagi:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 9040000
    miyagi,iwate:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 27100000
    miyagi,yamagata:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 2390000
    miyagi,fukushima:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 10400000
    miyagi,niigata:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 3770000
    fukushima,yamagata:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 1300000
    niigata,yamagata:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 7540000
```
</details>

`ac_transmission`の制約条件で送電容量を設定している。

## 次回までの課題

* 上記のコードを然るべき場所にコピペして実行してみる。
* コピペのコードでは供給が需要を満たしていないので、発電所を足したりして需要を満たす。
* モデルをOperarional modeで実行してみて結果を比較する。[ヒント](https://calliope.readthedocs.io/en/stable/user/advanced_features.html#operational-mode)
* 蓄電池や水素タンクなど、何か一つ電気を貯められる技術を導入する。[ヒント](https://calliope.readthedocs.io/en/stable/user/tutorials_01_national.html#storage-technologies)