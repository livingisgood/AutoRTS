### Server端

* 用NetMessage类逐步替换NetPacket来发送数据

  ```
  // 旧方式
  NetPacket packet;
  packet.setUInt16(0);
  packet.setUInt16(消息编号);
  // 填充数据
  packet.updateHeader_02();
  send(packet.getBuffer(), packet.getLength());
  ```

  ```
  // 替换为
  NetMessage message(消息编号);
  message << 数据;
  send(message);
  ```

  NetMessage 类会检查包体长度是否超过限制. 并且无需手动更新包头.

* 用BinaryReader类逐步替换 SafeGetXXX 宏来读取数据

  ```
  BinaryReader reader(buf, len);
  reader >> 数据;
  if(reader.isInvalid())
  {
  	处理解析错误,例如 return
  }
  ```

  reader类可以按值传递

  如果调用过 reader的任何读取方法, 那么在 reader的最后一次读取后会强制要求进行一次 isInvalid检查. 

  如果遗漏检查会在debug配置下产生assert中断

* pack(1) 的结构体可以被NetMessage和BinaryReader以流操作符默认发送.  为了防止误用, 这些结构体需继承 BytesPacked, 显式表明要按sizeof的方式发送

  ```
  struct BattleParticipant : BytesPacked
  {
      PlayerQueueData mQueueData;
      PlayerBattleData mBattleData;
  };
  
  struct BattleLocalResult : BytesPacked
  {
      int8_t mWinnerTeam {-1};
      int32_t mEndFrame {0};
  };
  ```

  非定长的消息结构体, 可以在结构体定义处提供inline的读写函数

  ```
  struct MatchAddress
  {
      int32_t mMatchID;
      int32_t mBattleServerID;
      std::wstring mBattleServerAddress;
      int32_t mBattleServerPort;
  };
  
  inline NetMessage& operator<< (NetMessage& message, const MatchAddress& v)
  {
      message << v.mMatchID;
      message << v.mBattleServerID;
      message << v.mBattleServerAddress;
      message << v.mBattleServerPort;
      return message;
  }
  ```

* 客户端和服务器交换数据的结构体不再定义在SharedStructures.h中, 而是各自定义在各自的NetStructures.h中, 采用各自的命名规范

  SharedStructures.h不再使用





