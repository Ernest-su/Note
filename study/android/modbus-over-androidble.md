---
title: Android基于Modbus RTU协议通过蓝牙与蓝牙串口开发板通信
date: 2021-02-08 16:46:55
categories:
    - android
tags:
    - android
    - modbus
---
### Android基于Modbus RTU协议通过蓝牙与蓝牙串口开发板通信

[原文](https://blog.csdn.net/weixin_42644340/article/details/111504661)

[TOC]



# 前言

最近接到一位老师的项目，他们公司要求利用Android基于Modbus RTU协议通过蓝牙与蓝牙串口开发板通信，然后通过开发板控制硬件设备从而实现Android移动设备与硬件交互。所以今天我给大家分享下Android基于Modbus RTU协议的蓝牙通信。当然Android也可以直接通过串口利用Modbus RTU协议与开发板通信，或者基于Modbus TCP通信。至于非蓝牙非Modbus RTU协议通信，大家可以跳转到这位博主的博客：[Steven Jon](https://stevenjon.blog.csdn.net/)

# 实现原理

要想实现Android与开发板通信，那么必将涉及到协议问题。Android采用蓝牙协议通信，而开发板使用的是串口的协议(Modbus协议)，那么应该如何实现让采用两种不同协议通信的设备进行交互呢？那当然是两种协议的套接啦，接下来我将以Modbus RTU协议给大家介绍下实现原理：
(1) 在Android中我们不能直接将指令放在蓝牙帧的数据域内，因为这样不能满足Modbus RTU协议，因此开发板无法从蓝牙帧中正确解析出你的指令，所以我们应该先将指令数据按照Modbus RTU协议先进行指令封装，然后将封装好的Modbus RTU协议指令帧整个放在蓝牙帧的数据域内。这样当开发板获取蓝牙帧时会自动获取数据域内的数据，这时候将获取到含有指令的Modbus RTU协议帧，然后从含有指令的Modbus RTU协议帧中解析出指令操作。
(2) 当然开发板要想从蓝牙帧中自动提取数据域中的数据也是有条件的，那就是在开发板上必须加上串口转蓝牙接口。
这个串口转蓝牙接口就是实现两种协议套接的核心。它的作用是如果获取到的是Android发出的蓝牙帧时自动提取数据区域内的Modbus RTU协议帧，而如果收到的是开发板发出的Modbus RTU协议帧时则自动将它封装到蓝牙帧中。都是文字的话大家是不是很难理解，那么接下来上原理图：

套接原理图：
![在这里插入图片描述](..\images\modbus_over_android_ble01.png)
功能原理图：
![在这里插入图片描述](..\images\modbus_over_android_ble2.png)看完实现原理接下来介绍最重要的代码实现。

# 在Android上实现Modbus RTU Master

在Android中针对Modbus RTU消息帧没有现成的组装模块，因此需要自己写。

## 组装Modbus RTU消息帧

这里是将输入的指令转换成对应的Modbus RTU协议的帧。

```javascript
     /**
     * 组装Modbus RTU消息帧
     * @param slave 从站地址号
     * @param function_code 功能码
     * @param starting_address 读取寄存器起始地址 / 写入寄存器地址 / 写入寄存器起始地址
     * @param quantity_of_x 读取寄存器个数 / 写入寄存器个数
     * @param output_value 需要写入单个寄存器的数值
     * @param output_values 需要写入多个寄存器的数组
     * @return 将整个消息帧转成byte[]
     * @throws ModbusError Modbus错误
     */
    synchronized private byte[] execute(int slave, int function_code, int starting_address, int quantity_of_x,
                                        int output_value, int[] output_values) throws ModbusError {
        //检查参数是否符合协议规定
        if (slave < 0 || slave > 0xff) {
            throw new ModbusError(ModbusErrorType.ModbusInvalidArgumentError, "Invalid slave " + slave);
        }
        if (starting_address < 0 || starting_address > 0xffff) {
            throw new ModbusError(ModbusErrorType.ModbusInvalidArgumentError, "Invalid starting_address " + starting_address);
        }
        if (quantity_of_x < 1 || quantity_of_x > 0xff) {
            throw new ModbusError(ModbusErrorType.ModbusInvalidArgumentError, "Invalid quantity_of_x " + quantity_of_x);
        }

        // 构造request
        ByteArrayWriter request = new ByteArrayWriter();
        //写入从站地址号
        request.writeInt8(slave);
        //根据功能码组装数据区
        //如果为读取寄存器指令
        if (function_code == ModbusFunction.READ_COILS || function_code == ModbusFunction.READ_DISCRETE_INPUTS
                || function_code == ModbusFunction.READ_INPUT_REGISTERS || function_code == ModbusFunction.READ_HOLDING_REGISTERS) {
            request.writeInt8(function_code);
            request.writeInt16(starting_address);
            request.writeInt16(quantity_of_x);

        } else if (function_code == ModbusFunction.WRITE_SINGLE_COIL || function_code == ModbusFunction.WRITE_SINGLE_REGISTER) {//写单个寄存器指令
            if (function_code == ModbusFunction.WRITE_SINGLE_COIL)
                if (output_value != 0) output_value = 0xff00;//如果为线圈寄存器（写1时为 FF 00,写0时为00 00）
            request.writeInt8(function_code);
            request.writeInt16(starting_address);
            request.writeInt16(output_value);

        } else if (function_code == ModbusFunction.WRITE_COILS) {//写多个线圈寄存器
            request.writeInt8(function_code);
            request.writeInt16(starting_address);
            request.writeInt16(quantity_of_x);

            //计算写入字节数
            int writeByteCount = (quantity_of_x / 8) + 1;/// 满足关系-> (w /8) + 1
            //写入数量 == 8 ,则写入字节数为1
            if (quantity_of_x % 8 == 0) {
                writeByteCount -= 1;
            }
            request.writeInt8(writeByteCount);

            int index = 0;
            //如果写入数据数量 > 8 ，则需要拆分开来
            int start = 0;//数组开始位置
            int end = 7;//数组结束位置
            int[] splitData = new int[8];
            //循环写入拆分数组，直到剩下最后一组 元素个数 <= 8 的数据
            while (writeByteCount > 1) {
                writeByteCount--;
                int sIndex = 0;
                for (index = start; index <= end; index++){
                    splitData [sIndex++] = output_values[index];
                }
                //数据反转 对于是否要反转要看你传过来的数据，如果高低位顺序正确则不用反转
                splitData = reverseArr(splitData);
                //写入拆分数组
                request.writeInt8(toDecimal(splitData));
                start = index;
                end += 8;
            }
            //写入最后剩下的数据
            int last = quantity_of_x - index;
            int[] tData = new int[last];
            System.arraycopy(output_values, index, tData, 0, last);
            //数据反转 对于是否要反转要看你传过来的数据，如果高低位顺序正确则不用反转
            tData = reverseArr(tData);
            request.writeInt8(toDecimal(tData));
        } else if (function_code == ModbusFunction.WRITE_HOLDING_REGISTERS) {//写多个保持寄存器
            request.writeInt8(function_code);
            request.writeInt16(starting_address);
            request.writeInt16(quantity_of_x);
            request.writeInt8(2 * quantity_of_x);
            //写入数据
            for (int v : output_values) {
                request.writeInt16(v);
            }
        } else {
            throw new ModbusError(ModbusErrorType.ModbusFunctionNotSupportedError, "Not support function " + function_code);
        }
        byte[] bytes = request.toByteArray();
        //计算CRC校验码
        int crc = CRC16.compute(bytes);
        request.writeInt16Reversal(crc);
        bytes = request.toByteArray();
        return bytes;
    }
```

## ModbusErrorType

定义Modbus通讯错误类型

```javascript
/**
 * 常见的Modbus通讯错误
 */
public enum ModbusErrorType {
    ModbusError,
    ModbusFunctionNotSupportedError,
    ModbusDuplicatedKeyError,
    ModbusMissingKeyError,
    ModbusInvalidBlockError,
    ModbusInvalidArgumentError,
    ModbusOverlapBlockError,
    ModbusOutOfBlockError,
    ModbusInvalidResponseError,
    ModbusInvalidRequestError,
    ModbusTimeoutError
}
```

## ModbusError

```javascript
public class ModbusError extends Exception {
    private int code;

    public ModbusError(int code, String message) {
        super(!TextUtils.isEmpty(message) ? message : "Modbus Error: Exception code = " + code);
        this.code = code;
    }

    public ModbusError(int code) {
        this(code, null);
    }

    public ModbusError(ModbusErrorType type, String message) {
        super(type.name() + ": " + message);
    }

    public ModbusError(String message) {
        super(message);
    }

    public int getCode() {
        return this.code;
    }
}
```

## ModbusFunction

功能码定义

```javascript
/**
 * 功能码（十进制显示）
 */
public class ModbusFunction {

    //读线圈寄存器
    public static final int READ_COILS = 1;

    //读离散输入寄存器
    public static final int READ_DISCRETE_INPUTS = 2;

    //读保持寄存器
    public static final int READ_HOLDING_REGISTERS = 3;

    //读输入寄存器
    public static final int READ_INPUT_REGISTERS = 4;

    //写单个线圈寄存器
    public static final int WRITE_SINGLE_COIL = 5;

    //写单个保持寄存器
    public static final int WRITE_SINGLE_REGISTER = 6;

    //写入多个线圈寄存器
    public static final int WRITE_COILS = 15;

    //写入多个保持寄存器
    public static final int WRITE_HOLDING_REGISTERS = 16;
}
```

## ByteArrayWriter

```javascript
public class ByteArrayWriter extends ByteArrayOutputStream {
    public ByteArrayWriter() {
        super();
    }

    public void writeInt8(byte b)
    {
        this.write(b);
    }

    public void writeInt8(int b)
    {
        this.write((byte)b);
    }

    public void writeInt16(int n) {
        byte[] bytes = ByteUtil.fromInt16(n);
        this.write(bytes, 0, bytes.length);
    }

    public void writeInt16Reversal(int n){
        byte[] bytes=ByteUtil.fromInt16Reversal(n);
        this.write(bytes,0,bytes.length);
    }

    public void writeInt32(int n) {
        byte[] bytes = ByteUtil.fromInt32(n);
        this.write(bytes, 0, bytes.length);
    }

    public void writeBytes(byte[] bs,int len){
        this.write(bs,0,len);
    }

}
```

## ByteUtil

```javascript
public class ByteUtil {

    public static String toHexString(byte[] input, String separator) {
        if (input==null) return null;

        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < input.length; i++) {
            if (separator != null && sb.length() > 0) {
                sb.append(separator);
            }
            String str = Integer.toHexString(input[i] & 0xff);
            if (str.length() == 1) str = "0" + str;
            sb.append(str);
        }
        return sb.toString();
    }

    public static String toHexString(byte[] input) {
        return toHexString(input, " ");
    }

    public static byte[] fromInt32(int input){
        byte[] result=new byte[4];
        result[3]=(byte)(input >> 24 & 0xFF);
        result[2]=(byte)(input >> 16 & 0xFF);
        result[1]=(byte)(input >> 8 & 0xFF);
        result[0]=(byte)(input & 0xFF);
        return result;
    }

    public static byte[] fromInt16(int input){
        byte[] result=new byte[2];
        result[0]=(byte)(input >> 8 & 0xFF);
        result[1]=(byte)(input & 0xFF);
        return result;
    }

    public static byte[] fromInt16Reversal(int input){
        byte[] result=new byte[2];
        result[1]=(byte)(input>>8&0xFF);
        result[0]=(byte)(input&0xFF);
        return result;
    }

}
```

## CRC16

```javascript
public class CRC16 {
    private static final byte[] crc16_tab_h = { (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40,
            (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x01, (byte) 0xC0, (byte) 0x80,
            (byte) 0x41, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0,
            (byte) 0x80, (byte) 0x41, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x00,
            (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41,
            (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x00, (byte) 0xC1, (byte) 0x81,
            (byte) 0x40, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0,
            (byte) 0x80, (byte) 0x41, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01,
            (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41,
            (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80,
            (byte) 0x41, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x00, (byte) 0xC1,
            (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x00,
            (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41,
            (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x00, (byte) 0xC1, (byte) 0x81,
            (byte) 0x40, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0,
            (byte) 0x80, (byte) 0x41, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x00,
            (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41,
            (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x00, (byte) 0xC1, (byte) 0x81,
            (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x01, (byte) 0xC0,
            (byte) 0x80, (byte) 0x41, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x00,
            (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41,
            (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80,
            (byte) 0x41, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x00, (byte) 0xC1,
            (byte) 0x81, (byte) 0x40, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01,
            (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41,
            (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80,
            (byte) 0x41, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x00, (byte) 0xC1,
            (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x00,
            (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41,
            (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x00, (byte) 0xC1, (byte) 0x81,
            (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x00, (byte) 0xC1,
            (byte) 0x81, (byte) 0x40, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x01,
            (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41,
            (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40, (byte) 0x00, (byte) 0xC1, (byte) 0x81,
            (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x00, (byte) 0xC1,
            (byte) 0x81, (byte) 0x40, (byte) 0x01, (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x01,
            (byte) 0xC0, (byte) 0x80, (byte) 0x41, (byte) 0x00, (byte) 0xC1, (byte) 0x81, (byte) 0x40 };

    private static final byte[] crc16_tab_l = { (byte) 0x00, (byte) 0xC0, (byte) 0xC1, (byte) 0x01,
            (byte) 0xC3, (byte) 0x03, (byte) 0x02, (byte) 0xC2, (byte) 0xC6, (byte) 0x06, (byte) 0x07,
            (byte) 0xC7, (byte) 0x05, (byte) 0xC5, (byte) 0xC4, (byte) 0x04, (byte) 0xCC, (byte) 0x0C,
            (byte) 0x0D, (byte) 0xCD, (byte) 0x0F, (byte) 0xCF, (byte) 0xCE, (byte) 0x0E, (byte) 0x0A,
            (byte) 0xCA, (byte) 0xCB, (byte) 0x0B, (byte) 0xC9, (byte) 0x09, (byte) 0x08, (byte) 0xC8,
            (byte) 0xD8, (byte) 0x18, (byte) 0x19, (byte) 0xD9, (byte) 0x1B, (byte) 0xDB, (byte) 0xDA,
            (byte) 0x1A, (byte) 0x1E, (byte) 0xDE, (byte) 0xDF, (byte) 0x1F, (byte) 0xDD, (byte) 0x1D,
            (byte) 0x1C, (byte) 0xDC, (byte) 0x14, (byte) 0xD4, (byte) 0xD5, (byte) 0x15, (byte) 0xD7,
            (byte) 0x17, (byte) 0x16, (byte) 0xD6, (byte) 0xD2, (byte) 0x12, (byte) 0x13, (byte) 0xD3,
            (byte) 0x11, (byte) 0xD1, (byte) 0xD0, (byte) 0x10, (byte) 0xF0, (byte) 0x30, (byte) 0x31,
            (byte) 0xF1, (byte) 0x33, (byte) 0xF3, (byte) 0xF2, (byte) 0x32, (byte) 0x36, (byte) 0xF6,
            (byte) 0xF7, (byte) 0x37, (byte) 0xF5, (byte) 0x35, (byte) 0x34, (byte) 0xF4, (byte) 0x3C,
            (byte) 0xFC, (byte) 0xFD, (byte) 0x3D, (byte) 0xFF, (byte) 0x3F, (byte) 0x3E, (byte) 0xFE,
            (byte) 0xFA, (byte) 0x3A, (byte) 0x3B, (byte) 0xFB, (byte) 0x39, (byte) 0xF9, (byte) 0xF8,
            (byte) 0x38, (byte) 0x28, (byte) 0xE8, (byte) 0xE9, (byte) 0x29, (byte) 0xEB, (byte) 0x2B,
            (byte) 0x2A, (byte) 0xEA, (byte) 0xEE, (byte) 0x2E, (byte) 0x2F, (byte) 0xEF, (byte) 0x2D,
            (byte) 0xED, (byte) 0xEC, (byte) 0x2C, (byte) 0xE4, (byte) 0x24, (byte) 0x25, (byte) 0xE5,
            (byte) 0x27, (byte) 0xE7, (byte) 0xE6, (byte) 0x26, (byte) 0x22, (byte) 0xE2, (byte) 0xE3,
            (byte) 0x23, (byte) 0xE1, (byte) 0x21, (byte) 0x20, (byte) 0xE0, (byte) 0xA0, (byte) 0x60,
            (byte) 0x61, (byte) 0xA1, (byte) 0x63, (byte) 0xA3, (byte) 0xA2, (byte) 0x62, (byte) 0x66,
            (byte) 0xA6, (byte) 0xA7, (byte) 0x67, (byte) 0xA5, (byte) 0x65, (byte) 0x64, (byte) 0xA4,
            (byte) 0x6C, (byte) 0xAC, (byte) 0xAD, (byte) 0x6D, (byte) 0xAF, (byte) 0x6F, (byte) 0x6E,
            (byte) 0xAE, (byte) 0xAA, (byte) 0x6A, (byte) 0x6B, (byte) 0xAB, (byte) 0x69, (byte) 0xA9,
            (byte) 0xA8, (byte) 0x68, (byte) 0x78, (byte) 0xB8, (byte) 0xB9, (byte) 0x79, (byte) 0xBB,
            (byte) 0x7B, (byte) 0x7A, (byte) 0xBA, (byte) 0xBE, (byte) 0x7E, (byte) 0x7F, (byte) 0xBF,
            (byte) 0x7D, (byte) 0xBD, (byte) 0xBC, (byte) 0x7C, (byte) 0xB4, (byte) 0x74, (byte) 0x75,
            (byte) 0xB5, (byte) 0x77, (byte) 0xB7, (byte) 0xB6, (byte) 0x76, (byte) 0x72, (byte) 0xB2,
            (byte) 0xB3, (byte) 0x73, (byte) 0xB1, (byte) 0x71, (byte) 0x70, (byte) 0xB0, (byte) 0x50,
            (byte) 0x90, (byte) 0x91, (byte) 0x51, (byte) 0x93, (byte) 0x53, (byte) 0x52, (byte) 0x92,
            (byte) 0x96, (byte) 0x56, (byte) 0x57, (byte) 0x97, (byte) 0x55, (byte) 0x95, (byte) 0x94,
            (byte) 0x54, (byte) 0x9C, (byte) 0x5C, (byte) 0x5D, (byte) 0x9D, (byte) 0x5F, (byte) 0x9F,
            (byte) 0x9E, (byte) 0x5E, (byte) 0x5A, (byte) 0x9A, (byte) 0x9B, (byte) 0x5B, (byte) 0x99,
            (byte) 0x59, (byte) 0x58, (byte) 0x98, (byte) 0x88, (byte) 0x48, (byte) 0x49, (byte) 0x89,
            (byte) 0x4B, (byte) 0x8B, (byte) 0x8A, (byte) 0x4A, (byte) 0x4E, (byte) 0x8E, (byte) 0x8F,
            (byte) 0x4F, (byte) 0x8D, (byte) 0x4D, (byte) 0x4C, (byte) 0x8C, (byte) 0x44, (byte) 0x84,
            (byte) 0x85, (byte) 0x45, (byte) 0x87, (byte) 0x47, (byte) 0x46, (byte) 0x86, (byte) 0x82,
            (byte) 0x42, (byte) 0x43, (byte) 0x83, (byte) 0x41, (byte) 0x81, (byte) 0x80, (byte) 0x40 };

    /**
     * 计算CRC16校验
     *
     * @param data
     *            需要计算的数组
     * @return CRC16校验值
     */
    public static int compute(byte[] data) {
        return compute(data, 0, data.length);
    }

    /**
     * 计算CRC16校验
     *
     * @param data
     *            需要计算的数组
     * @param offset
     *            起始位置
     * @param len
     *            长度
     * @return CRC16校验值
     */
    public static int compute(byte[] data, int offset, int len) {
        return compute(data, offset, len, 0xffff);
    }

    /**
     * 计算CRC16校验
     *
     * @param data
     *            需要计算的数组
     * @param offset
     *            起始位置
     * @param len
     *            长度
     * @param preval
     *            之前的校验值
     * @return CRC16校验值
     */
    public static int compute(byte[] data, int offset, int len, int preval) {
        int ucCRCHi = (preval & 0xff00) >> 8;
        int ucCRCLo = preval & 0x00ff;
        int iIndex;
        for (int i = 0; i < len; ++i) {
            iIndex = (ucCRCLo ^ data[offset + i]) & 0x00ff;
            ucCRCLo = ucCRCHi ^ crc16_tab_h[iIndex];
            ucCRCHi = crc16_tab_l[iIndex];
        }
        int result=((ucCRCHi & 0x00ff) << 8) | (ucCRCLo & 0x00ff) & 0xffff;
        return result;
    }
}
```

## ModbusRtuMaster完整代码

这个类主要的功能就是将输入的指令转换成符合ModbusRtu协议的指令并返回，之后Android只需将返回的byte[]类型的值用蓝牙发送即可。

```javascript
import java.util.Formatter;

public class ModbusRtuMaster {


    /**
     * 组装Modbus RTU消息帧
     *
     * @param slave            从站地址号
     * @param function_code    功能码
     * @param starting_address 读取寄存器起始地址 / 写入寄存器地址 / 写入寄存器起始地址
     * @param quantity_of_x    读取寄存器个数 / 写入寄存器个数
     * @param output_value     需要写入单个寄存器的数值
     * @param output_values    需要写入多个寄存器的数组
     * @return 将整个消息帧转成byte[]
     * @throws ModbusError Modbus错误
     */
    synchronized private byte[] execute(int slave, int function_code, int starting_address, int quantity_of_x,
                                        int output_value, int[] output_values) throws ModbusError {
        //检查参数是否符合协议规定
        if (slave < 0 || slave > 0xff) {
            throw new ModbusError(ModbusErrorType.ModbusInvalidArgumentError, "Invalid slave " + slave);
        }
        if (starting_address < 0 || starting_address > 0xffff) {
            throw new ModbusError(ModbusErrorType.ModbusInvalidArgumentError, "Invalid starting_address " + starting_address);
        }
        if (quantity_of_x < 1 || quantity_of_x > 0xff) {
            throw new ModbusError(ModbusErrorType.ModbusInvalidArgumentError, "Invalid quantity_of_x " + quantity_of_x);
        }

        // 构造request
        ByteArrayWriter request = new ByteArrayWriter();
        //写入从站地址号
        request.writeInt8(slave);
        //根据功能码组装数据区
        //如果为读取寄存器指令
        if (function_code == ModbusFunction.READ_COILS || function_code == ModbusFunction.READ_DISCRETE_INPUTS
                || function_code == ModbusFunction.READ_INPUT_REGISTERS || function_code == ModbusFunction.READ_HOLDING_REGISTERS) {
            request.writeInt8(function_code);
            request.writeInt16(starting_address);
            request.writeInt16(quantity_of_x);

        } else if (function_code == ModbusFunction.WRITE_SINGLE_COIL || function_code == ModbusFunction.WRITE_SINGLE_REGISTER) {//写单个寄存器指令
            if (function_code == ModbusFunction.WRITE_SINGLE_COIL)
                if (output_value != 0) output_value = 0xff00;//如果为线圈寄存器（写1时为 FF 00,写0时为00 00）
            request.writeInt8(function_code);
            request.writeInt16(starting_address);
            request.writeInt16(output_value);

        } else if (function_code == ModbusFunction.WRITE_COILS) {//写多个线圈寄存器
            request.writeInt8(function_code);
            request.writeInt16(starting_address);
            request.writeInt16(quantity_of_x);

            //计算写入字节数
            int writeByteCount = (quantity_of_x / 8) + 1;/// 满足关系-> (w /8) + 1
            //写入数量 == 8 ,则写入字节数为1
            if (quantity_of_x % 8 == 0) {
                writeByteCount -= 1;
            }
            request.writeInt8(writeByteCount);

            int index = 0;
            //如果写入数据数量 > 8 ，则需要拆分开来
            int start = 0;//数组开始位置
            int end = 7;//数组结束位置
            int[] splitData = new int[8];
            //循环写入拆分数组，直到剩下最后一组 元素个数 <= 8 的数据
            while (writeByteCount > 1) {
                writeByteCount--;
                int sIndex = 0;
                for (index = start; index <= end; index++) {
                    splitData[sIndex++] = output_values[index];
                }
                //数据反转 对于是否要反转要看你传过来的数据，如果高低位顺序正确则不用反转
                splitData = reverseArr(splitData);
                //写入拆分数组
                request.writeInt8(toDecimal(splitData));
                start = index;
                end += 8;
            }
            //写入最后剩下的数据
            int last = quantity_of_x - index;
            int[] tData = new int[last];
            System.arraycopy(output_values, index, tData, 0, last);
            //数据反转 对于是否要反转要看你传过来的数据，如果高低位顺序正确则不用反转
            tData = reverseArr(tData);
            request.writeInt8(toDecimal(tData));
        } else if (function_code == ModbusFunction.WRITE_HOLDING_REGISTERS) {//写多个保持寄存器
            request.writeInt8(function_code);
            request.writeInt16(starting_address);
            request.writeInt16(quantity_of_x);
            request.writeInt8(2 * quantity_of_x);
            //写入数据
            for (int v : output_values) {
                request.writeInt16(v);
            }
        } else {
            throw new ModbusError(ModbusErrorType.ModbusFunctionNotSupportedError, "Not support function " + function_code);
        }
        byte[] bytes = request.toByteArray();
        //计算CRC校验码
        int crc = CRC16.compute(bytes);
        request.writeInt16Reversal(crc);
        bytes = request.toByteArray();
        return bytes;
    }

    /**
     * 读多个线圈寄存器
     *
     * @param slave          从站地址
     * @param startAddress   起始地址
     * @param numberOfPoints 读取线圈寄存器个数
     * @throws ModbusError Modbus错误
     */
    public byte[] readCoils(int slave, int startAddress, int numberOfPoints) throws ModbusError {
        byte[] sendBytes = execute(slave, ModbusFunction.READ_COILS, startAddress, numberOfPoints, 0, null);
        return sendBytes;
    }

    //读单个线圈寄存器
    public byte[] readCoil(int slave, int address) throws ModbusError {
        byte[] sendBytes = readCoils(slave, address, 1);
        return sendBytes;
    }

    /**
     * 读多个保持寄存器
     *
     * @param slave          从站地址
     * @param startAddress   起始地址
     * @param numberOfPoints 读取保持寄存器个数
     * @throws ModbusError Modbus错误
     */
    public byte[] readHoldingRegisters(int slave, int startAddress, int numberOfPoints) throws ModbusError {
        byte[] sendBytes = execute(slave, ModbusFunction.READ_HOLDING_REGISTERS, startAddress, numberOfPoints, 0, null);

        return sendBytes;
    }

    //读单个保持寄存器
    public byte[] readHoldingRegister(int slave, int address) throws ModbusError {
        byte[] sendBytes = readHoldingRegisters(slave, address, 1);
        return sendBytes;
    }


    /**
     * 读多个输入寄存器
     *
     * @param slave          从站地址
     * @param startAddress   起始地址
     * @param numberOfPoints 读取输入寄存器个数
     * @throws ModbusError Modbus错误
     */
    public byte[] readInputRegisters(int slave, int startAddress, int numberOfPoints) throws ModbusError {
        byte[] sendBytes = execute(slave, ModbusFunction.READ_INPUT_REGISTERS, startAddress, numberOfPoints, 0, null);
        return sendBytes;
    }

    //读单个输入寄存器
    public byte[] readInputRegister(int slave, int address) throws ModbusError {
        byte[] sendBytes = readInputRegisters(slave, address, 1);
        return sendBytes;
    }

    /**
     * 读多个离散输入寄存器
     *
     * @param slave          从站地址
     * @param startAddress   起始地址
     * @param numberOfPoints 读取离散输入寄存器个数
     * @throws ModbusError Modbus错误
     */
    public byte[] readDiscreteInputs(int slave, int startAddress, int numberOfPoints) throws ModbusError {
        byte[] sendBytes = execute(slave, ModbusFunction.READ_DISCRETE_INPUTS, startAddress, numberOfPoints, 0, null);
        return sendBytes;
    }

    //读单个离散输入寄存器
    public byte[] readDiscreteInput(int slave, int address) throws ModbusError {
        byte[] sendBytes = readDiscreteInputs(slave, address, 1);
        return sendBytes;
    }

    /**
     * 写单个线圈寄存器
     *
     * @param slave   从站地址
     * @param address 写入寄存器地址
     * @param value   写入值（true/false)
     * @throws ModbusError Modbus错误
     */
    public byte[] writeSingleCoil(int slave, int address, boolean value) throws ModbusError {
        byte[] sendBytes = execute(slave, ModbusFunction.WRITE_SINGLE_COIL, address, 1, value ? 1 : 0, null);
        return sendBytes;
    }

    /**
     * 写单个保持寄存器
     *
     * @param slave   从站地址
     * @param address 写入寄存器地址
     * @param value   写入值
     * @throws ModbusError Modbus错误
     */
    public byte[] writeSingleRegister(int slave, int address, int value) throws ModbusError {
        byte[] sendBytes = execute(slave, ModbusFunction.WRITE_SINGLE_REGISTER, address, 1, value, null);
        return sendBytes;
    }

    /**
     * 写入多个保持寄存器
     *
     * @param slave   从站地址
     * @param address 写入寄存器地址
     * @param sCount  写入寄存器个数
     * @param data    写入数据
     * @throws ModbusError
     */
    public byte[] writeHoldingRegisters(int slave, int address, int sCount, int[] data) throws ModbusError {
        byte[] sendBytes = execute(slave, ModbusFunction.WRITE_HOLDING_REGISTERS, address, sCount, 0, data);
        return sendBytes;
    }

    /**
     * 写入多个位
     *
     * @param slave   从站地址
     * @param address 写入寄存器地址
     * @param bCount  写入寄存器个数
     * @param data    写入数据{1,0}
     * @throws ModbusError
     */
    public byte[] writeCoils(int slave, int address, int bCount, int[] data) throws ModbusError {
        byte[] sendBytes = execute(slave, ModbusFunction.WRITE_COILS, address, bCount, 0, data);
        return sendBytes;
    }


    //将数组反转
    private int[] reverseArr(int[] arr) {
        int[] tem = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            tem[i] = arr[arr.length - 1 - i];
        }
        return tem;
    }

    /**
     * 字节转换为 16 进制字符串
     *
     * @param b 字节
     * @return Hex 字符串
     */
    static String byte2Hex(byte b) {
        StringBuilder hex = new StringBuilder(Integer.toHexString(b));
        if (hex.length() > 2) {
            hex = new StringBuilder(hex.substring(hex.length() - 2));
        }
        while (hex.length() < 2) {
            hex.insert(0, "0");
        }
        return hex.toString();
    }


    /**
     * 字节数组转换为 16 进制字符串
     *
     * @param bytes 字节数组
     * @return Hex 字符串
     */
    static String bytes2Hex(byte[] bytes) {
        Formatter formatter = new Formatter();
        for (byte b : bytes) {
            formatter.format("%02x ", b);
        }
        String hash = formatter.toString();
        formatter.close();
        return hash;
    }

    //将int[1,0,0,1,1,0]数组转换为十进制数据
    private int toDecimal(int[] data) {
        int result = 0;
        if (data != null) {
            StringBuilder sData = new StringBuilder();
            for (int d : data) {
                sData.append(d);
            }
            try {
                result = Integer.parseInt(sData.toString(), 2);
            } catch (NumberFormatException e) {
                result = -1;
            }

        }
        return result;
    }
}
```

# 在Android上实现用蓝牙传递Modbus Rtu协议帧

今天分享的是Android如何基于Modbus RTU协议与蓝牙串口开发板通信，至于Android如何搜索蓝牙设备，如何连接我这就不多做介绍了，网上有很多博主做了一大堆了。接下来来看看我们写好的ModbusRtuMaster怎么使用吧。

```javascript
//首先在你的activity中定义一个ModbusRtuMaster类型
private ModbusRtuMaster modbusRtuMaster;
//然后在activity中的onCreate方法里初始化ModbusRtuMaster
modbusRtuMaster = new ModbusRtuMaster();
//最后利用ModbusRtuMaster对输入的指令进行协议封装并接收返回结果
//这时候results就是一个符合ModbusRtu协议的消息针了
byte[] results = modbusRtuMaster.writeHoldingRegisters(Rs485, Rsaddress, Hvalue.length, Hvalue);
//接下来可以把results用蓝牙发给你的蓝牙串口开发板了
private final OutputStream mmOutStream;
mmOutStream.write(results)
```

# 字节数组转换为 16 进制字符串的方法

这里我再提下ModbusRtuMaster里面的bytes2Hex方法，如果你想把转换后的指令或者是串口返回的数据显示在Android设备上，那么必须将其转化为Hex类型的16进制字符串，如果不转换成Hex那么显示出来将是乱码等，因为那些数据是btye类型，然后就是这里不能使用
new String(byte[])
去将byte数组转换成string，否则将显示成符号字母，请务必按照我在ModbusRtuMaster内写的bytes2Hex将字节数组转换为 16 进制字符串，再输出显示。

```javascript
static String bytes2Hex(byte[] bytes) {
        Formatter formatter = new Formatter();
        for (byte b : bytes) {
            formatter.format("%02x ", b);
        }
        String hash = formatter.toString();
        formatter.close();
        return hash;
    }
```

如何使用bytes2Hex

```javascript
String result = modbusRtuMaster.bytes2Hex(results);
```

# 最终效果

最后让我们来看看使用ModbusRtuMaster返回帧通过蓝牙传给蓝牙串口开发板的效果吧！(蓝色代表发送指令，绿色代表接收开发板返回的数据)
![在这里插入图片描述](..\images\modbus_over_android_ble03.png)

以上就是《Android基于ModbusRtu协议通过蓝牙与蓝牙串口开发板通信》的全部内容了，我也是第一次使用Android去通信开发板与硬件交互。这里非常感谢另一位博主的文章 [Steven Jon](https://stevenjon.blog.csdn.net/)，我也是借鉴了他博客中的一篇关于ModbusRtuMaster的文章才实现通信的，对其他通信方法感兴趣的小伙伴可以点击去Steven Jon大佬的博客继续学习。