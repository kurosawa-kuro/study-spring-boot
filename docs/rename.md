以下は **テンプレリポジトリに同梱しておく “rename.sh” 完整版** です。
`groupId` （パッケージ階層）と `artifactId / projectName` （フォルダ名・アプリ名）を一発で差し替え、
Maven／Gradle どちらでも動くようにしています。

> ✱ 使い方
>
> ```bash
> ./rename.sh com.acme billing-service
> ```
>
> ︙実行後に **src/main/java/com/acme/billingservice/** へディレクトリを自動生成し、
> `com.template.*` だった import／package 行をすべて置換します。

```bash
#!/usr/bin/env bash
# rename.sh  --------------------------------------------------------------
# 引数:
#   $1 = 新しい groupId (例: com.acme)
#   $2 = 新しい artifactId / application-name (例: billing-service)
# ------------------------------------------------------------------------

set -euo pipefail

if [[ $# -ne 2 ]]; then
  echo "Usage: ./rename.sh <groupId> <artifactId>"
  exit 1
fi

NEW_GROUP=$1                # com.acme
NEW_ARTIFACT=$2             # billing-service
TEMPLATE_GROUP="com.template"
TEMPLATE_ART="starter-template"

echo "🔄  Renaming '${TEMPLATE_GROUP}.${TEMPLATE_ART}'  →  '${NEW_GROUP}.${NEW_ARTIFACT}'"

# 0) calc paths
SRC_JAVA_DIR="src/main/java"
SRC_TEST_DIR="src/test/java"
NEW_GROUP_PATH=$(echo "${NEW_GROUP}" | tr '.' '/')

# 1) Replace text tokens in files (pom.xml, build.gradle*, *.java, *.kt, yml etc.)
echo "📝  Replacing text tokens..."
# targets: everything except binary files
grep -rl --exclude-dir=.git --exclude-dir=build --exclude=rename.sh \
  -e "${TEMPLATE_GROUP}" -e "${TEMPLATE_ART}" . \
  | xargs sed -i \
    -e "s|${TEMPLATE_GROUP}|${NEW_GROUP}|g" \
    -e "s|${TEMPLATE_ART}|${NEW_ARTIFACT}|g"

# 2) Move package directories (Java/Kotlin)
echo "📂  Moving package directory..."
mkdir -p "${SRC_JAVA_DIR}/${NEW_GROUP_PATH}/${NEW_ARTIFACT//-/_}"
mkdir -p "${SRC_TEST_DIR}/${NEW_GROUP_PATH}/${NEW_ARTIFACT//-/_}"

rsync -a --remove-source-files \
  "${SRC_JAVA_DIR}/${TEMPLATE_GROUP//./\/}/${TEMPLATE_ART//-/_}/" \
  "${SRC_JAVA_DIR}/${NEW_GROUP_PATH}/${NEW_ARTIFACT//-/_}/"

rsync -a --remove-source-files \
  "${SRC_TEST_DIR}/${TEMPLATE_GROUP//./\/}/${TEMPLATE_ART//-/_}/" \
  "${SRC_TEST_DIR}/${NEW_GROUP_PATH}/${NEW_ARTIFACT//-/_}/" 2>/dev/null || true

# delete residual empty dirs
find "${SRC_JAVA_DIR}" "${SRC_TEST_DIR}" -type d -empty -delete

# 3) Rename the Spring application class if存在
APP_CLASS="${SRC_JAVA_DIR}/${NEW_GROUP_PATH}/${NEW_ARTIFACT//-/_}/StarterTemplateApplication.java"
if [[ -f "${APP_CLASS}" ]]; then
  NEW_APP_CLASS="${START=StarterTemplateApplication; echo ${APP_CLASS/StarterTemplate/${NEW_ARTIFACT//-/_^}}"
fi

# 4) Gradle settings or Maven artifactId replacement are already done by sed.

# 5) Update application name in properties/yml
if grep -q "^spring.application.name=" src/main/resources/application.properties 2>/dev/null; then
  sed -i "s|spring.application.name=.*|spring.application.name=${NEW_ARTIFACT}|g" src/main/resources/application.properties
fi
if grep -q "spring.application.name:" src/main/resources/application.yml 2>/dev/null; then
  sed -i "s|spring.application.name:.*|spring.application.name: ${NEW_ARTIFACT}|g" src/main/resources/application.yml
fi

echo "✅  Done. Package root: ${NEW_GROUP}.${NEW_ARTIFACT//-/_}"
echo "👉  NEXT:"
echo "   1. mvn test           # or ./gradlew test"
echo "   2. git grep com.template  # should return nothing"
echo "   3. git init && git add -A && git commit -m 'initial commit'"
```

### スクリプトのポイント

| 処理                          | 詳細                                                                 |                                                                   |
| --------------------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------- |
| **テキスト一括置換**                | \`grep -rl                                                         | sed -i`で`pom.xml / build.gradle\[.kts] / \*.java / \*.yml\` を一網打尽 |
| **パッケージ階層の移動**              | `rsync --remove-source-files` を使い *中身を移してから空ディレクトリ削除*              |                                                                   |
| **artifactId のハイフン→アンダーバー** | Java のパスでは `-` が使えないため `billing-service` → `billing_service` に自動変換 |                                                                   |
| **プロパティ名更新**                | `spring.application.name` を同時に書き換え readinessProbe で名前ズレ事故を防止       |                                                                   |

---

これを **テンプレリポジトリのルート** に置いて、
README に「`./rename.sh <groupId> <artifactId>` を叩くだけ」と書いておけば、
**マイクロサービス増殖フェーズでも “名前置換地獄” から解放** されます。

さらなる自動化（GitHub Actions で post-generate フックなど）が必要なら、また相談してください！
