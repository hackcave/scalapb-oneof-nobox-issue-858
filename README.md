# sbt-native-packager-docker-issue-1348

Original bug report at [https://github.com/scalapb/ScalaPB/issues/858](https://github.com/scalapb/ScalaPB/issues/858)

To reproduce the bug:

```bash
git clone https://github.com/hackcave/scalapb-oneof-nobox-issue-858.git
cd scalapb-oneof-nobox-issue-858 && sbt compile
```

It fails with this error:
```log
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:21:18: value += is not a member of Int
[error]   Expression does not convert to assignment because:
[error]     value serializedSize is not a member of Option[example.protobuf.duration.Seconds]
[error]     value serializedSize is not a member of Option[example.protobuf.duration.Seconds]
[error]     expansion: __size = __size.+(1.+(_root_.com.google.protobuf.CodedOutputStream.computeUInt32SizeNoTag(__value.<serializedSize: error>)).<$plus: error>(__value.<serializedSize: error>))
[error]           __size += 1 + _root_.com.google.protobuf.CodedOutputStream.computeUInt32SizeNoTag(__value.serializedSize) + __value.serializedSize
[error]                  ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:28:18: value += is not a member of Int
[error]   Expression does not convert to assignment because:
[error]     value serializedSize is not a member of Option[example.protobuf.duration.Minutes]
[error]     value serializedSize is not a member of Option[example.protobuf.duration.Minutes]
[error]     expansion: __size = __size.+(1.+(_root_.com.google.protobuf.CodedOutputStream.computeUInt32SizeNoTag(__value.<serializedSize: error>)).<$plus: error>(__value.<serializedSize: error>))
[error]           __size += 1 + _root_.com.google.protobuf.CodedOutputStream.computeUInt32SizeNoTag(__value.serializedSize) + __value.serializedSize
[error]                  ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:47:42: value serializedSize is not a member of Option[example.protobuf.duration.Seconds]
[error]           _output__.writeUInt32NoTag(__v.serializedSize)
[error]                                          ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:48:15: value writeTo is not a member of Option[example.protobuf.duration.Seconds]
[error]           __v.writeTo(_output__)
[error]               ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:55:42: value serializedSize is not a member of Option[example.protobuf.duration.Minutes]
[error]           _output__.writeUInt32NoTag(__v.serializedSize)
[error]                                          ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:56:15: value writeTo is not a member of Option[example.protobuf.duration.Minutes]
[error]           __v.writeTo(_output__)
[error]               ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:63:78: not found: value seconds
[error]     def withSeconds(__v: example.protobuf.duration.Seconds): Duration = copy(seconds = __v)
[error]                                                                              ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:66:78: not found: value minutes
[error]     def withMinutes(__v: example.protobuf.duration.Minutes): Duration = copy(minutes = __v)
[error]                                                                              ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:80:44: value toPMessage is not a member of Option[example.protobuf.duration.Seconds]
[error]         case 1 => durationOptional.seconds.toPMessage.getOrElse(_root_.scalapb.descriptors.PEmpty)
[error]                                            ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:81:44: value toPMessage is not a member of Option[example.protobuf.duration.Minutes]
[error]         case 2 => durationOptional.minutes.toPMessage.getOrElse(_root_.scalapb.descriptors.PEmpty)
[error]                                            ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:66:9: method withMinutes is defined twice;
[error]   the conflicting method withMinutes was defined at line 65:9
[error]     def withMinutes(__v: example.protobuf.duration.Minutes): Duration = copy(minutes = __v)
[error]         ^
[error] scalapb-oneof-nobox-issue-858/target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala:63:9: method withSeconds is defined twice;
[error]   the conflicting method withSeconds was defined at line 62:9
[error]     def withSeconds(__v: example.protobuf.duration.Seconds): Duration = copy(seconds = __v)
[error]         ^
[error] 12 errors found
[error] (Compile / compileIncremental) Compilation failed
```

Generated code has following conflicting methods:

In file `target/scala-2.12/src_managed/main/scalapb/example/protobuf/duration/Duration.scala`:
```scala
...
    def withSeconds(__v: example.protobuf.duration.Seconds): Duration = copy(durationOptional = example.protobuf.duration.Duration.DurationOptional.Seconds(__v))
    def withSeconds(__v: example.protobuf.duration.Seconds): Duration = copy(seconds = __v)
...
    def withMinutes(__v: example.protobuf.duration.Minutes): Duration = copy(durationOptional = example.protobuf.duration.Duration.DurationOptional.Minutes(__v))
    def withMinutes(__v: example.protobuf.duration.Minutes): Duration = copy(minutes = __v)
...
```

which leads to compilation failure along with a set of similar errors due to type confusion.
