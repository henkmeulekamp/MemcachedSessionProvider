# Memcached Session Provider

This is a highly available, high performance ASP.NET session state store provider using Memcached. 

Features:

* Handles Memcached node failures
* No session locking per request

### Handling node failures
In a pool of memcached nodes, sessions are distributed equally across all the nodes. Additionally, for every session, a backup 
copy is stored on a secondary memcached node. This is similar to the way [Memcached session manager for Tomcat](https://code.google.com/p/memcached-session-manager/) is implemented.

Example: In  a pool of 2 memcached nodes, if a session `S1` gets stored on memcached node `M1`, the backup session `bak:S1` is stored 
on node `M2`. If node `M1` goes down, session is not lost. It will be retrived from the `M2` node without any interruption. 
Similarly, `M1` acts as backup node for `M2`, in case `M2` goes down. 
```
<M1>     <M2>
 S1		  S2
bak:S2	 bak:S1
```
Note that if only 1 memcached node is configured then there is no backup. 

### No session locking
[Session locking in ASP.NET](http://msdn.microsoft.com/en-us/library/ms178587.aspx) can cause a few 
[performance problems](http://stackoverflow.com/questions/3629709/i-just-discovered-why-all-asp-net-websites-are-slow-and-i-am-trying-to-work-out). 
This custom implementation of Session provider does not lock any Session. This should not be a problem in most cases. But if 
you need locked, consistent data as part of the session, you can implement a lock/concurrency check in your application, 
or use a different Memcached provider (see [here](https://github.com/enyim/memcached-providers) and [here](http://memcachedproviders.codeplex.com/)).

## Requirements
You'll need .NET Framework 3.5 or later to use the precompiled binaries. To build client, you'll need Visual Studio 2012.

## Install
In your web project, include the assembly via [NuGet Package](https://www.nuget.org/packages/MemcachedSessionProvider/). 

## Web.config
This library uses the [Enyim Memcached client](https://github.com/enyim/EnyimMemcached). Make the following changes in 
the web.config file for Enyim client
```xml
<configuration>
	<configSections>
		<sectionGroup name="sessionManagement">
			<section name="memcached" type="Enyim.Caching.Configuration.MemcachedClientSection, Enyim.Caching" />
		</sectionGroup>
	</configSections>

	<sessionManagement>
		<memcached protocol="Binary">
			<servers>
				<!-- make sure you use the same ordering of nodes in every configuration you have -->
				<add address="ip address" port="port number" />
				<add address="ip address" port="port number" />
			</servers>
			<locator type="MemcachedSessionProvider.SessionNodeLocator, MemcachedSessionProvider" />
		</memcached>
	</enyim.com>

</configuration>
```
#### memcached/locator
The `memcached/locator` is used to map objects to servers in the pool. Replace the default implementation with the 
type `MemcachedSessionProvider.SessionNodeLocator, MemcachedSessionProvider`. This handles the session backup. 

See here for [more configuration options for Enyim Memcached](https://github.com/enyim/EnyimMemcached/wiki/MemcachedClient-Configuration)

Also make the following change in web.config to use the custom Session State provider
```xml
<configuration>

	<system.web>
		
		<sessionState customProvider="Memcached" mode="Custom">
			<providers>
				<add name="Memcached" type="MemcachedSessionProvider.SessionProvider, MemcachedSessionProvider" />
			</providers>
		</sessionState>

	</system.web>

</configuration>
```

## Questions?
If you have questions, bugs reports, or feature requests, please submit them via the [Issue Tracker](https://github.com/rohita/MemcachedSessionProvider/issues).

## Reference
This implementation is based on the [sample provided by Microsoft](http://msdn.microsoft.com/en-us/library/ms178588.aspx).
