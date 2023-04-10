# 硬件测试用例

## 1、测试工具

[测试工具代码仓库](https://gitee.com/opencloudos-stream/oc-hct)

## 2、测试项目


<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-0lax{text-align:left;vertical-align:top}
</style>
<table class="tg">
<thead>
  <tr>
    <th class="tg-0lax"><span style="font-weight:bold;font-style:normal;color:#000">硬件分类</span></th>
    <th class="tg-0lax"><span style="font-weight:bold;font-style:normal;color:#000">硬件子类</span></th>
    <th class="tg-0lax"><span style="font-weight:bold;font-style:normal;color:#000">测试策略</span></th>
    <th class="tg-0lax"><span style="font-weight:bold;font-style:normal;color:#000">测试分类</span></th>
    <th class="tg-0lax"><span style="font-weight:bold;font-style:normal;color:#000">测试项</span></th>
    <th class="tg-0lax"><span style="font-weight:bold;font-style:normal;color:#000">用例</span></th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-0lax" rowspan="2"><span style="font-weight:normal;font-style:normal;color:#000">整机</span></td>
    <td class="tg-0lax" rowspan="2"><span style="font-weight:normal;font-style:normal;color:#000">服务器整机　</span></td>
    <td class="tg-0lax" rowspan="2"><span style="font-weight:normal;font-style:normal;color:#000">1、不同厂商、不同型号的服务器需要要分别做兼容性测试</span><br><span style="font-weight:normal;font-style:normal;color:#000">2、服务器主板拓扑（如增减CPU、内存、外设等）发生变化</span><br><span style="font-weight:normal;font-style:normal;color:#000">3、服务器最大规格扩容（如内存、存储）</span><br><span style="font-weight:normal;font-style:normal;color:#000">4、服务器推荐使用典型配置，内存、存储推荐使用最大规格做兼容性测试　</span></td>
    <td class="tg-0lax" rowspan="2"><span style="font-weight:normal;font-style:normal;color:#000">整机测试　</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">自动识别设备并执行测试项</span></td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">函数调用栈测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">system.backtrace</span></td>
  </tr>
  <tr>
    <td class="tg-0lax" rowspan="16"><span style="font-weight:normal;font-style:normal;color:#000">部件</span></td>
    <td class="tg-0lax" rowspan="4"><span style="font-weight:normal;font-style:normal;color:#000">cpu</span></td>
    <td class="tg-0lax" rowspan="4"><span style="font-weight:normal;font-style:normal;color:#000">1、CPU不同的架构、微架构和代次需要分别做兼容性测试</span></td>
    <td class="tg-0lax" rowspan="4"><span style="font-weight:normal;font-style:normal;color:#000">功能测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">CPU识别</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">cpu.power</span><br><span style="font-weight:normal;font-style:normal;color:#000">cpu.list</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">CPU热插拔</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">cpu.hotplug</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">计算和调度</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">cpu.benchmark</span><br><span style="font-weight:normal;font-style:normal;color:#000">cpu.schedule</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">浮点测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">cpu.calculate</span></td>
  </tr>
  <tr>
    <td class="tg-0lax" rowspan="3"><span style="font-weight:normal;font-style:normal;color:#000">memory</span></td>
    <td class="tg-0lax" rowspan="3"><span style="font-weight:normal;font-style:normal;color:#000">1、不同品牌、不同型号、不同速率需要分别做兼容性测试，推荐使用最大容量进行测试</span></td>
    <td class="tg-0lax" rowspan="3"><span style="font-weight:normal;font-style:normal;color:#000">功能测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">内存识别</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">memory.list</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">内存热插拔</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">memory.hotplug</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">内存读写测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">memory.allocate</span></td>
  </tr>
  <tr>
    <td class="tg-0lax" rowspan="2"><span style="font-weight:normal;font-style:normal;color:#000">gpu</span></td>
    <td class="tg-0lax" rowspan="2"><span style="font-weight:normal;font-style:normal;color:#000">1、不同品牌、不同型号需要分别做兼容性测试</span></td>
    <td class="tg-0lax" rowspan="2"><span style="font-weight:normal;font-style:normal;color:#000">功能测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">显卡设备识别</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">gpu.list</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">计算加速</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">gpu.vendor.nvidia</span></td>
  </tr>
  <tr>
    <td class="tg-0lax" rowspan="3"><span style="font-weight:normal;font-style:normal;color:#000">storage</span></td>
    <td class="tg-0lax" rowspan="3"><span style="font-weight:normal;font-style:normal;color:#000">1、不同品牌、不同型号的RAID卡需要分别做兼容性测试</span><br><span style="font-weight:normal;font-style:normal;color:#000">2、兼容性测试需要覆盖主流配置</span><br><span style="font-weight:normal;font-style:normal;color:#000">3、不同品牌、不同介质（HDD、SSD、NVME等）、不同型号需要分别做兼容性测试，推荐使用最大容量进行测试</span></td>
    <td class="tg-0lax" rowspan="3"><span style="font-weight:normal;font-style:normal;color:#000">功能测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">存储设备识别</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">storage.list</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">典型文件系统测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">storage.mount</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">存储功能测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">storage.rw</span></td>
  </tr>
  <tr>
    <td class="tg-0lax" rowspan="3"><span style="font-weight:normal;font-style:normal;color:#000">network</span></td>
    <td class="tg-0lax" rowspan="3"><span style="font-weight:normal;font-style:normal;color:#000">1、不同品牌、不同型号需要分别做兼容性测试，需要使用最大速率进行测试</span></td>
    <td class="tg-0lax" rowspan="3"><span style="font-weight:normal;font-style:normal;color:#000">功能测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">网络设备识别</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">network.list</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">网络连通性测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">network.connect</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">协议测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">network.tcp</span><br><span style="font-weight:normal;font-style:normal;color:#000">network.udp</span></td>
  </tr>
  <tr>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">其他设备</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">1、不同品牌、不同型号需要分别做兼容性测试，具体根据设备特点单独分析决定</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">功能测试</span></td>
    <td class="tg-0lax"><span style="font-weight:normal;font-style:normal;color:#000">根据具体设备验证</span></td>
    <td class="tg-0lax"></td>
  </tr>
</tbody>
</table>