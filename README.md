HowToUseGradle
==============

# Gradle を使ってパッケージを作るための設定について (OSX 限定)


## 環境設定

### gradle 導入

以下コマンドで 1.6 が導入されます (2013.10.29 現在)。

    $ brew install gradle
  
### Eclipse での設定

Eclipse 上で Gradle による Android パッケージの Build を行なうためには ADT 22 および SDK Tools 22 の導入が必要となっていますので、まずそちらの導入を行なっておいて下さい。また、ADT 22 導入後は以下の不具合が出ることが確認されていますので、対処を行なう必要があるかもしれません。

- libs にライブラリを置いて参照している場合、Build Path の設定の変更を行なう必要があります
- SDK Build Tools の導入を行なう必要があります

ADT 22 の導入により、External Tools による gradle に実行が可能となる模様です。

### External Tools Configurations

Eclipse 上で Gradle による Android パッケージの Build を行なうには、Run -> External Tools -> External Tools Configurations より Gradle の実行に関する設定を盛り込んでおく必要があります。
エントリを作成するには上記項目を選択した後に、左ペインに存在する「Program」を右クリックし、新規作成を選択します。あとは以下に示す項目に適切な値を入力して「Apply」ボタンをクリックしておいて下さい。

* Name には適切な名前を入力しておいて下さい
* Main Tab
 * Location には /usr/local/bin/gradle と入力します
 * Working Directory には ${project\_loc} と入力します (Variables.. からの選択が可能)
 * Arguments には −-daemon ${string\_prompt} と入力します
 
* Environment Tab
 * JAVA\_OPT という variable を新規に追加し、「-Dgroovy.source.encoding=UTF-8 -Dfile.encoding=UTF-8」という記述を Value に追加します
 + ANDROID\_HOME という variable を新規に追加し、SDK のディレクトリの絶対パス記述を Value に記述します


## build.gradle の雛形

署名済みの .apk を出力するための build.gradle および gradle.properties が以下となります。また、配布パッケージに署名するための keystore は /User/hoge/.android/release.keystore にあるものとします。

build.gradle

    buildscript {
        repositories {
            mavenCentral()
         }
         dependencies {
             classpath 'com.android.tools.build:gradle:0.5.+'
         }
      }
      apply plugin: 'android'
      
      dependencies {
          compile fileTree(dir: 'libs', include: '*.jar')
      }
      
    android {
       compileSdkVersion 15
       buildToolsVersion "18.1.1"
   
       sourceSets {
           main {
               manifest.srcFile 'AndroidManifest.xml'
               java.srcDirs = ['src']
               resources.srcDirs = ['src']
               aidl.srcDirs = ['src']
               renderscript.srcDirs = ['src']
               res.srcDirs = ['res']
               assets.srcDirs = ['assets']
           }
   
           // Move the tests to tests/java, tests/res, etc...
           instrumentTest.setRoot('tests')
   
           // Move the build types to build-types/<type>
           // For instance, build-types/debug/java, build-types/debug/AndroidManifest.xml, ...
           // This moves them out of them default location under src/<type>/... which would
           // conflict with src/ being used by the main source set.
           // Adding new build types or product flavors should be accompanied
           // by a similar customization.
           debug.setRoot('build-types/debug')
           release.setRoot('build-types/release')
       }
       
       signingConfigs {
           debug {
               storeFile file("debug.keystore")
           }
           
           myConfig
       }
         
       if (project.hasProperty('storeFile')) {
           android.signingConfigs.myConfig.storeFile = file(storeFile)
       }
       if (project.hasProperty('storePassword')) {
           android.signingConfigs.myConfig.storePassword = storePassword
       }
       if (project.hasProperty('keyAlias')) {
           android.signingConfigs.myConfig.keyAlias = keyAlias
       }
       if (project.hasProperty('keyPassword')) {
           android.signingConfigs.myConfig.keyPassword = keyPassword
       }
       
       buildTypes {
           debug {
               signingConfig signingConfigs.debug
           }
           release {
               signingConfig signingConfigs.myConfig
           }
       }
    }

gradle.properties

    storeFile=/Users/hoge/.android/release.keystore
    storePassword=hoge
    keyAlias=hoge
    keyPassword=hoge

## 実行

ここでは release な apk を出力する処理に限定して記述します。

### Eclipse から起動

Run -> External Tools -> External Tools Configrations から Gradle 実行の構成を表示し、右下にある Run ボタンを click することでオプション入力なダイアログが表示されます。clean aR と入力して OK ボタンをクリックすることにより release な apk が build/apk 配下に出力されます。

### コマンドラインから起動

以下で実行が可能です。

    $ export ANDROID_HOME=/Users/hoge/dev/android/sdk
    $ export JAVA_OPTS="-Dgroovy.source.encoding=UTF-8 -Dfile.encoding=UTF-8"
    $ gradle clean aR

