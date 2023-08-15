---
title: mybatis动态sql
tags: mybatis
categories: mybatis动态sql学习
abbrlink: 5787
date: 2023-08-07 00:57:15
---

# if 标签

可以判断是否符合条件，再进行赋值查询

```xml
<select id="xxx" parameterType="User" resultMap="baseMap">
	select * 
    from tb_user
    where 1=1
    <if test="userName != null">
        and user_name=#{userName}
    </if>
    <if test="realName != null">
        and real_name=#{realName}
    </if>
    <if test="age > 0">
        and age=#{age}
    </if>
    <if test="id != null">
        and id=#{id}
    </if>
</select>
```

# where 标签

在写**`where`**的时候，为了保证 **`SQL`** 的的语法正确，在有**`and`**的情况下，可以用**`1=1`**表达式，若不想这么写的话，可以用**`where`**标签

```xml
<select id="xxx" parameterType="User" resultMap="baseMap">
	select * 
	from tb_user
    <where>
        <if test="userName != null">
            and user_name=#{userName}
        </if>
        <if test="realName != null">
            and real_name=#{realName}
        </if>
        <if test="age > 0">
            and age=#{age}
        </if>
        <if test="id != null">
            and id=#{id}
        </if>
    </where>
</select>
```

# choose 标签

**`choose`**相当于Java中的**`switch`**语句，若第一个不成立，则执行第二个，若第二个也不成立，则执行最后的**`otherwise`**标签里面的内容

```xml
<select id="xxx" parameterType="User" resultMap="baseMap">
	select * 
	from tb_user
    <where>
		<choose>
        	<when test="userName != null">
            	and user_name = #{userName}
            </when>
            <when test="realName != null">
            	and real_name = #{realName}
            </when>
            <otherwise>
            	order by id desc
            </otherwise>
        </choose>
    </where>
</select>
```

# set 标签

可以去掉多余的 **`,`**  ，避免了**`SQL`**报错

```xml
<update id="xxx" parameterType="User">
	update 
    	tb_user
	<set>
    	<if test="userName != null">
        	user_name = #{userName},
        </if>
        <if test="realName != null">
            real_name = #{realName},
        </if>
        <if test="age > 0">
            age = #{age},
        </if>
    </set>
    where
    	id = #{id}
</update>
```

# trim 标签

**`trim`**标签可以完成 **`where`** 或者 **`set`** 标签的功能，**`prefix`**表示前缀为**where**标签，**`prefixOverrides`**表示去掉前面多余的**and**

* 第一种

```xml
<select id="xxx" parameterType="User" resultMap="baseMap">
	select *
	from tb_user
	<trim prefix="where" prefixOverrides="AND | OR">
        <if test="userName != null">
        	and user_name = #{userName}
        </if>
        <if test="realName != null">
            and real_name = #{realName}
        </if>
        <if test="age > 0">
            and age = #{age}
        </if>
    </trim>
</select>
```

**`suffixOverrides`**表示去掉后面多余的 **`,`**

* 第二种

```xml
<update id="xxx" parameterType="User">
	update
    	tb_user
	<trim prefix="set" suffixOverrides=",">
    	<if test="userName != null">
        	user_name = #{userName},
        </if>
        <if test="realName != null">
            real_name = #{realName},
        </if>
        <if test="age > 0">
            age = #{age},
        </if>
    </trim>
    where
    	id = #{id}
</update>
```

# forEach 标签

**`forEach`** 标签可以对集合进行遍历，**`open`**表示开始的括号，**`close`**表示最后的括号，**`separator`**表示用 **`,`** 分割，**`item`**表示每一个元素

```xml
<select id="xxx" parameterType="list" resultMap="baseMap">
	select *
	from tb_user
	<where>
        <if test="ids != null">
        	id in
            <foreach collection="ids" open="(" close=")" separator="," item="id">
                #{id}
            </foreach>
        </if>
    </where>
</select>
```

# bind 标签

**`bind`**标签允许在**`OGNL`**表达式以外创建一个变量，将该变量绑定到上下文中，可以供后续使用

```xml
<select id="xxx" parameterType="User" resultMap="baseMap">
    <!-- <bind name="userNameLike" value="'%'+_parameter.getUserName()+'%'"/> -->
    <bind name="userNameLike" value="'%' + userName + '%'"/>
	select *
	from tb_user
	<where>
        <if test="userName != null">
        	user_name like #{userNameLike}
        </if>
    </where>
</select>
```



