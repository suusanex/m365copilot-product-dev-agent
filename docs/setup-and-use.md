# 製品向け Skill の導入・利用ガイド

共通リポジトリは解析手順と設定欄を提供します。製品担当者は SharePoint 上の案内と資料、閲覧権限、ソースコードの所在と対象版、ソースを調査できる外部エージェントを用意します。

## 1. 利用条件を確認する

仕事用アカウントで通常の Microsoft 365 Copilot Chat を開きます。利用者に個人 Skill と仕事用コンテンツの参照を利用できる Microsoft 365 Copilot ライセンスが割り当てられ、**Agents & Skills** を開けることを確認します。Microsoft の案内では個人 Skill は Premium 向けです。製品資料への閲覧権限も利用者のアカウントで確認します。[Skill の利用条件](https://support.microsoft.com/en-gb/microsoft-365-copilot/create-edit-skills-copilot) · [ライセンスの選択肢](https://learn.microsoft.com/en-us/microsoft-365/copilot/microsoft-365-copilot-licensing)

Microsoft Purview Information Barriers を使用するテナントでは Copilot Chat の Skill がサポートされません。利用条件と `SKILL.md` の仕様は Microsoft Learn で確認できます。[Copilot Chat Skill の仕様](https://learn.microsoft.com/en-us/plugins/build-copilot-chat-skills)

対象テナントで、Skill の実行と SharePoint 資料を使った解析が既存ライセンス内で行え、追加の Copilot クレジットを必要としないことを確認します。必要なら管理者に割り当てや課金条件を確認し、追加クレジットを要求される経路は今回の運用に使いません。[Microsoft 365 Copilot のライセンスと費用](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/cost-considerations)

## 2. 非公開環境に製品向けコピーを作る

1. SharePoint で、既存資料のうちどれをどの場面・対象版で読むかを説明する案内への直接リンクを用意します。既存の文書やページを案内として使えます。案内から資料を見つけられるよう、必要に応じて資料へのリンクや識別できる名称を示します。形式・フォルダ構成・資料の管理方法は製品側で選びます。
2. [共通 SKILL.md](../skills/product-incident-analysis/SKILL.md) を製品側の非公開作業場所へコピーします。「製品設定」の `{{PRODUCT_NAME}}` と `{{KNOWLEDGE_GUIDE_URL}}` を置換します。任意の補足には、その製品で必要な版の表記やログの相関キーなどを記入できます。
3. 複数の製品向け Skill を登録する場合は、先頭の `name` を製品ごとに区別できる名前にします。`name` は半角英小文字・数字・ハイフンで 1〜64 文字です。先頭・末尾のハイフンと連続するハイフンは使用できません。`description` に該当製品を記載する際は 1〜1,024 文字の YAML 文字列として記述します。先頭の YAML 区切りと `SKILL.md` のファイル名は維持します。設定を埋めたファイル、資料、実ログは製品側で管理します。

## 3. Copilot Chat に登録・更新する

1. Copilot Chat の **Agents & Skills** を開き、**Add → Upload skill** から製品向けに編集した単体の `SKILL.md` をアップロードします。名称と説明を確認して保存し、登録した Skill を開き直して製品設定を確認します。表示はロールアウトにより異なる場合があります。[Microsoft の登録手順](https://support.microsoft.com/en-gb/microsoft-365-copilot/create-edit-skills-copilot)
2. 更新時は非公開のコピーを先に編集します。**Agents & Skills** で登録済み Skill を開き、同じ内容に更新して保存します。新しいチャットで選び直し、対象製品と参照先が更新されていることを確認します。

## 4. 解析を始める

1. 新しい通常チャットで対象 Skill を明示的に選びます。「このトレースから障害原因の候補を検討してください」と依頼し、共有可能なログ、症状と期待動作、発生時刻とタイムゾーン、環境と対象版を分かる範囲で渡します。
2. 回答に出た資料名・該当箇所とログの根拠を確認します。案内または関連資料が取得できない場合は、閲覧権限とチャットのソース設定を確認します。チャットの参照メニューで対象ファイルを明示して再依頼するか、利用環境で参照できる形式の資料や共有可能な抜粋を渡します。[Copilot Chat でのファイル指定](https://support.microsoft.com/en-us/microsoft-365-copilot/refer-to-specific-files-and-more-in-microsoft-365-copilot)
3. 実装調査を求められたら、共有範囲を確認し、対象リポジトリとログに対応する版・コミットを指定して、調査プロンプトを外部エージェントに手動で渡します。結果を元のチャットに戻し、どの仮説が支持・反証されたか、残る確認が何かを確かめます。

## 5. 製品側で一往復を確認する

次の架空例を目安に、実際の SharePoint 資料と対象版が分かるログで試します。

1. 架空製品「帆灯」の版 2.4 で、同一相関 ID の順に `enqueue.accepted`、`worker.lease-expired` が出たとします。案内が指す版 2.4 の処理資料では受付後にワーカーへ処理を割り当てます。資料とログを照合し、処理中のリース期限切れと、受け取る前の割り当て失敗を候補に挙げます。
2. 外部エージェントに、配備版に対応するコミットで `worker.lease-expired` が処理開始の前後どちらで出得るかを調査させます。結果の例は「対象コミット `abc1234` の `src/Worker.cs:86` では、処理開始済みの分岐でのみ記録する」です。
3. 元のチャットで結果を照合し、割り当て前の失敗を候補から外せる理由と、期限切れの原因を決めるには処理開始からの経過時間などが必要なことを確認します。
4. 資料が取得できない場合や対象版に合わない報告でも、未確認箇所と次に必要な情報が示されるかを確認します。

実際の導入時は、Skill が選択され、意図した SharePoint の案内と資料を読めたか、資料で仮説を立てられたか、実装証拠で仮説を更新できたか、どこで手動の資料指定や情報補足が必要だったかを記録します。追加クレジットの要否も対象テナントで確認します。架空例は手順と受け渡す情報の確認に使います。
