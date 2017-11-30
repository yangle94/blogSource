---
title: SpringCloud记录点
date: 2017-09-26 11:32:35
tags: SpringCloud
---

### 忽略网络接口

application.yml:

~~~ yml
spring:
  cloud:
    inetutils:
      ignoredInterfaces:
        - docker0
        - veth.*
~~~

代码位置:
org.springframework.cloud.commons.util.InetUtilsProperties

<!--more-->

~~~java
	/**
	 * List of Java regex expressions for network addresses that will be 	 * preferred.
	 */
	private List<String> preferredNetworks = new ArrayList<>();
~~~
这个类是一个一个网络工具类的配置文件属性类，由注释可得，preferredNetworks承载了所有需要进行忽略的网络连接名的正则表达式。查看其使用位置

~~~java
boolean ignoreInterface(String interfaceName) {
		for (String regex : this.properties.getIgnoredInterfaces()) {
			if (interfaceName.matches(regex)) {
				log.trace("Ignoring interface: " + interfaceName);
				return true;
			}
		}
		return false;
	}
~~~

查看ignoreInterface所调用的位置：

~~~java
public InetAddress findFirstNonLoopbackAddress() {
		InetAddress result = null;
		try {
			int lowest = Integer.MAX_VALUE;
			for (Enumeration<NetworkInterface> nics = NetworkInterface
					.getNetworkInterfaces(); nics.hasMoreElements();) {
				NetworkInterface ifc = nics.nextElement();
				if (ifc.isUp()) {
					log.trace("Testing interface: " + ifc.getDisplayName());
					if (ifc.getIndex() < lowest || result == null) {
						lowest = ifc.getIndex();
					}
					else if (result != null) {
						continue;
					}

					// @formatter:off
					if (!ignoreInterface(ifc.getDisplayName())) {
						for (Enumeration<InetAddress> addrs = ifc
								.getInetAddresses(); addrs.hasMoreElements();) {
							InetAddress address = addrs.nextElement();
							if (address instanceof Inet4Address
									&& !address.isLoopbackAddress()
									&& !ignoreAddress(address)) {
								log.trace("Found non-loopback interface: "
										+ ifc.getDisplayName());
								result = address;
							}
						}
					}
					// @formatter:on
				}
			}
		}
		catch (IOException ex) {
			log.error("Cannot get first non-loopback address", ex);
		}

		if (result != null) {
			return result;
		}

		try {
			return InetAddress.getLocalHost();
		}
		catch (UnknownHostException e) {
			log.warn("Unable to retrieve localhost");
		}

		return null;
	}
~~~

SpringCloud进行注册的时候会调用本地方法获得当前宿主机的网络信息，在*nix下（包括Mac），类似于ifconfig命令所显示的内容，会遍历其中的内容，所有网卡下到上，然后分别每个网卡内部的地址，保留最后一个非回环地址。

我在当某台Linux机器（宿主机）安装了docker的时候，会安装一个docker0的网桥，docker0的网桥会在宿主机的第一个，导致在宿主机上运行的springboot（dubbo也会）会拿到错误的IP信息，当把docker0网卡加入到这个里头之后，就会避免这个问题。

### 强制使用正则表达式中的地址以及仅使用站点本地地址

~~~yml
spring:
  cloud:
    inetutils:
      preferredNetworks:
        - 192.168
        - 10.0
~~~

此配置同样位于org.springframework.cloud.commons.util.InetUtilsProperties

~~~java
	/**
	 * Use only interfaces with site local addresses. See {@link InetAddress#isSiteLocalAddress()} for more details.
	 */
	private boolean useOnlySiteLocalInterfaces = false;

~~~

查看参数使用位置：

~~~java
boolean ignoreAddress(InetAddress address) {

		if (this.properties.isUseOnlySiteLocalInterfaces() && !address.isSiteLocalAddress()) {
			log.trace("Ignoring address: " + address.getHostAddress());
			return true;
		}
~~~

由此可见，当不设置的时候，useOnlySiteLocalInterfaces默认值为false，不管拿到的是公网地址还是内网地址，直接进行对下面正则的匹配，当正则不匹配并且地址不是以regex开头的，会返回true（注意方法名是ignoreAddress，true表示忽略，false表示不忽略），如果不匹配则返回false；当useOnlySiteLocalInterfaces设置为true时，如果此IP地址不是内网地址，则直接返回true，如果是内网地址，则进行正则匹配。