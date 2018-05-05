---
title: UE4打印大全
date: 2018-05-02 15:56:12
tags:
- UE4
categories:
- UE4
---


# 介绍

下面的代码取自我的GitHub项目 [<i class="fa fa-fw fa-github fa-2x"></i>**RealTimeServer**](https://github.com/no5ix/RealTimeServer)
都是自描述性的且项目无关的, 可以直接放心使用.

**. . .**<!-- more -->

# 使用例子

覆盖了以下几种场合 : 

- Example usage: A_LOG();

- Example usage: A_LOG_1( "Action!" );

- Example usage: A_LOG_2("Action!", "Cut!");

- Example usage: A_LOG_N("Action!", 88.f);

- Example usage: A_LOG_M("Action! %f, %d", 88.f, 88);



# 示例代码

``` c++ RealTimeSrvHelper.h
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#define ACTION_SHOW_DEBUG_SCREEN_MSG					false
#define ACTION_SHOW_DEBUG_OUTPUT_LOG					false
#define ACTION_SHOW_DEBUG_OUTPUT_LOG_EXTRA				false


//Current Class Name + Function Name where this is called!
#define STR_CUR_CLASS_FUNC							(FString(__FUNCTION__))

//Current Class where this is called!
#define STR_CUR_CLASS								(FString(__FUNCTION__).Left(FString(__FUNCTION__).Find(TEXT(":"))) )

//Current Function Name where this is called!
#define STR_CUR_FUNC								(FString(__FUNCTION__).Right(FString(__FUNCTION__).Len() - FString(__FUNCTION__).Find(TEXT("::")) - 2 ))

//Current Line Number in the code where this is called!
#define STR_CUR_LINE								(FString::FromInt(__LINE__))

//Current Class and Line Number where this is called!
#define STR_CUR_CLASS_LINE							(STR_CUR_CLASS + "(" + STR_CUR_LINE + ")")

//Current Class and Line Number where this is called!
#define STR_CUR_CLASS_FUNC_LINE						(STR_CUR_CLASS_FUNC + "(" + STR_CUR_LINE + ")")

//Current Function Signature where this is called!
#define STR_CUR_FUNCSIG								(FString(__FUNCSIG__))


////////////// Screen Message
// 	Gives you the Class name and exact line number where you print a message to yourself!


#define A_MSG(TimeToDisplay )								if (ACTION_SHOW_DEBUG_SCREEN_MSG) (GEngine->AddOnScreenDebugMessage(-1, (float)TimeToDisplay, FColor::Red, *(STR_CUR_CLASS_FUNC_LINE )) )

#define A_MSG_1(TimeToDisplay, StringParam1)                        if (ACTION_SHOW_DEBUG_SCREEN_MSG) (GEngine->AddOnScreenDebugMessage(-1, (float)TimeToDisplay, FColor::Red, *(STR_CUR_CLASS_FUNC_LINE + "  :  " + StringParam1)) )

#define A_MSG_2(TimeToDisplay, StringParam1, StringParam2)     			if (ACTION_SHOW_DEBUG_SCREEN_MSG) (GEngine->AddOnScreenDebugMessage(-1, (float)TimeToDisplay, FColor::Red, *(STR_CUR_CLASS_FUNC_LINE + "  :  " + StringParam1 + "      " + StringParam2)) )

//#define A_SCREENMSG_F(StringParam1, NumericalParam2)     		if (ACTION_SHOW_DEBUG_SCREEN_MSG) (GEngine->AddOnScreenDebugMessage(-1, (float)TimeToDisplay, FColor::Red, *(STR_CUR_CLASS_FUNC_LINE + "  :  " + StringParam1 + "      " + FString::SanitizeFloat(NumericalParam2))) )
#define A_MSG_N(TimeToDisplay, StringParam1, NumericalParam2)     		if (ACTION_SHOW_DEBUG_SCREEN_MSG) (GEngine->AddOnScreenDebugMessage(-1, (float)TimeToDisplay, FColor::Red, FString::Printf( TEXT("%s  :  %s    %f"), *STR_CUR_CLASS_FUNC_LINE, *FString(StringParam1), float(NumericalParam2) ) ) )

#define A_MSG_M(TimeToDisplay, FormatString, ...)     		if (ACTION_SHOW_DEBUG_SCREEN_MSG) (GEngine->AddOnScreenDebugMessage(-1, (float)TimeToDisplay, FColor::Red, FString::Printf( TEXT("%s  :  %s"), *STR_CUR_CLASS_FUNC_LINE, *FString::Printf(TEXT(FormatString), ##__VA_ARGS__ ) ) ) )



///////// UE LOG!

// Example usage: A_LOG();
#define	A_LOG() 		           					if (ACTION_SHOW_DEBUG_OUTPUT_LOG) UE_LOG(LogTemp, Warning, TEXT("%s"), *STR_CUR_CLASS_FUNC_LINE )

// Example usage: A_LOG_1( "Action!" );
#define A_LOG_1(StringParam1) 		           				if (ACTION_SHOW_DEBUG_OUTPUT_LOG) UE_LOG(LogTemp, Warning, TEXT("%s  :  %s"), *STR_CUR_CLASS_FUNC_LINE, *FString(StringParam1))

// Example usage: A_LOG_2("Action!", "Cut!");
#define A_LOG_2(StringParam1, StringParam2) 	       				if (ACTION_SHOW_DEBUG_OUTPUT_LOG) UE_LOG(LogTemp, Warning, TEXT("%s  :  %s     %s"), *STR_CUR_CLASS_FUNC_LINE, *FString(StringParam1), *FString(StringParam2))

// Example usage: A_LOG_N("Action!", 88.f);
#define A_LOG_N(StringParam1, NumericalParam2) 	       		if (ACTION_SHOW_DEBUG_OUTPUT_LOG) UE_LOG(LogTemp, Warning, TEXT("%s  :  %s    %f"), *STR_CUR_CLASS_FUNC_LINE, *FString(StringParam1), float(NumericalParam2) )

// 
#define A_LOG_M(FormatString, ...)     				if (ACTION_SHOW_DEBUG_OUTPUT_LOG) UE_LOG(LogTemp, Warning, TEXT("%s  :  %s"), *STR_CUR_CLASS_FUNC_LINE, *FString::Printf(TEXT(FormatString), ##__VA_ARGS__ ) )

class RealTimeSrvHelper
{
public:

	static void ScreenMsg( const FString& Msg );
	static void ScreenMsg( const FString& Msg, const FString& Msg2 );
	static void ScreenMsg( const FString& Msg, const float FloatValue );
};
```


``` c++ RealTimeSrvHelper.cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "RealTimeSrvHelper.h"


//ScreenMsg
void RealTimeSrvHelper::ScreenMsg( const FString& Msg )
{
	if (!ACTION_SHOW_DEBUG_SCREEN_MSG) return;
	GEngine->AddOnScreenDebugMessage( -1, 55.f, FColor::Red, *Msg );
}

void RealTimeSrvHelper::ScreenMsg( const FString& Msg, const FString& Msg2 )
{
	if (!ACTION_SHOW_DEBUG_SCREEN_MSG) return;
	GEngine->AddOnScreenDebugMessage( -1, 55.f, FColor::Red, FString::Printf( TEXT( "%s %s" ), *Msg, *Msg2 ) );
}

void RealTimeSrvHelper::ScreenMsg( const FString& Msg, const float Value )
{
	if (!ACTION_SHOW_DEBUG_SCREEN_MSG) return;
	GEngine->AddOnScreenDebugMessage( -1, 55.f, FColor::Red, FString::Printf( TEXT( "%s %f" ), *Msg, Value ) );
}


```