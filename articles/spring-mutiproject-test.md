---
title: "【Spring Boot】【Kotlin】別プロジェクトのテストパッケージを参照する"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# Spring Boot (Kotlin) で別プロジェクトのテストパッケージを参照する方法
SpringBoot(Kotlin)のマルチプロジェクトの構成で、project-aのtestから、project-bのtestを参照したいケースがあったのでメモ。

内容は以下のサンプル通り、project-aでtestのJarファイルを作成し、project-bで読み込みます。

```project-a/bundle.gradle.kts
configurations {
    create("test")
}

tasks.register<Jar>("testArchive") {
    archiveBaseName.set("project-a-test")
    // project-aからtestパッケージを出力する
    from(project.the<SourceSetContainer>()["test"].output)

    // project-aとproject-bに@SpringBootApplicationがある場合には、@SpringBootApplicationの重複エラーが発生してしまう。
    // project-aの@SpringBootApplicationが書いてあるファイルをxxx.classとした場合、以下で除外できる。
    // exculude(**/xxx.class)
}

// 他プロジェクトから"test"で読み込めるようにする
artifacts {
    add("test", tasks["testArchive"])
}
```

```projet-b/bundle.gradle.kts
dependencies {
    implementation(project(":project-a"))
    testImplementation(project(":project-a", "test"))
}
```

参考サイト
https://stackoverflow.com/questions/5644011/multi-project-test-dependencies-with-gradle/61682321#61682321