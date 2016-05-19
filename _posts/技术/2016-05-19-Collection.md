---
layout: post
title: 集合类封装
category: 技术
tags: Trans
keywords: 
description: 
---

# 集合类封装

  通常来讲，集合类是不可变对象。所以要时刻维护类作用范围使得其不对调用者无意暴露。
  一种方式是对调用者提供一些相关方法来代替直接调用，比如：

	addThing(Thing)
	removeThing(Thing)
	getThings() -返回一个不可变的集合

**例1**
	
	import java.util.*
	public final class SoccerTeam{
		
		public SoccerTeam(){
			...
		}		

		public void addPlayer(Player player){
			fPlayers.add(player);	
		}

		public void removePlayer(Player player){
			fPlayers.remove(player);
		}

		public Set<Player> getPlayer(){
			return Collections.unmodifiableSet(fPlayers);
		}

		...

		private Set<Player> fPlayers = new LinkedHashSet<>();
	}

	

**例2**
  *BaseballTeam*是一个直接暴露给调用者的例子。这是一个不正确的设计，因为这个集合类的内容可以被*BaseballTeam*和调用者同时修改。

	import java.util.*;
	public final class BaseballTeam{
		public BaseballTeam(){
			...
		}

		public void setPlayers(Set<Player> players){
			fPlayers = players;
		}

		public Set<Player> getPlayers(){
			return fPlayers;
		}
		
		...

		private Set<Player> fPlayers;
	}

------

[原帖地址](http://javapractices.com/topic/TopicAction.do?Id=173)

------