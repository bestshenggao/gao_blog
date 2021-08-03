# Animated 动画

React-native Animated 旨在以声明的形式来定义动画的输入与输出，在其中建立一个可配置的变化函数，然后使用start/stop方法来控制动画执行。

::: details 基础动画。opacity过度

```jsx
import React, { useEffect, useRef } from "react";
import { Animated, Text, View, StyleSheet } from "react-native";

export default () => {
    const fadeAnim = useRef(new Animated.Value(0)).current;
    useEffect(() => {
        Animated.timing(fadeAnim, {
            toValue: 1,
            duration: 5000
        }).start();
    }, []);
    return <View style={styles.container}>
        <Animated.View
            style={[
                styles.fadingContainer,
                {
                    opacity: fadeAnim
                }
            ]}
        >
            <Text style={styles.fadingText}>我出现了</Text>
        </Animated.View>
    </View>
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        alignItems: "center",
        justifyContent: "center"
    },
    fadingContainer: {
        paddingVertical: 8,
        paddingHorizontal: 16,
        backgroundColor: "powderblue"
    },
    fadingText: {
        fontSize: 28,
        textAlign: "center",
        margin: 10
    },
});
```

:::

::: details interpolate(插值动画)。多个属性同时改变

```jsx
import React, { useEffect, useRef } from 'react';
import { View, Animated, Easing, StyleSheet } from 'react-native';

export default () => {
    const animation = useRef(new Animated.Value(0)).current;
    useEffect(() => {
        Animated.timing(animation, {
            toValue: 1,
            duration: 1000,
            easing: Easing.linear
        }).start();
    }, []);

    return <View style={styles.container}>
        <Animated.View style={
            [
                {
                    opacity: animation,
                    transform: [{
                        rotateZ: animation.interpolate({
                            inputRange: [0, 1],
                            outputRange: ['0deg', '720deg']
                        })
                    }]
                }]}>
            <Animated.Text style={{
                fontSize: animation.interpolate({
                    inputRange: [0, 1],
                    outputRange: [10, 24]
                })
            }}>
                😄😄升高😄😄
            </Animated.Text>
        </Animated.View>
    </View>
}

const styles = StyleSheet.create({
    container: {
        width: '100%',
        height: '100%',
        justifyContent: 'center',
        alignItems: 'center'
    }
});
```

:::

::: details sequence(顺序执行动画)。目录淡入淡出

```jsx
import React, { useEffect, useRef } from 'react';
import { View, Animated, StyleSheet, Easing } from 'react-native';

export default () => {
    const animationArray = Array.from({ length: 3 }).map(() => new Animated.Value(0));
    const animations = useRef(animationArray).current;
    useEffect(() => {
        Animated.sequence(
            [
                // 向右移动
                ...animations.map(anim => {
                    return Animated.timing(anim, {
                        toValue: 1,
                        duration: 600,
                        easing: Easing.linear,
                        useNativeDriver: true
                    })
                }),
                // 延迟600毫秒
                Animated.delay(600),
                // 向左移动，返回原始状态
                ...animations.map(anim => {
                    return Animated.timing(anim, {
                        toValue: 0,
                        duration: 600,
                        easing: Easing.linear,
                        useNativeDriver: true
                    })
                }).reverse()
            ]
        ).start();
    })

    return (
        <View style={styles.box}>
            {
                animations.map((anim, index) => {
                    return (
                        <Animated.View
                            style={
                                [
                                    styles.list,
                                    styles[`list${index + 1}`],
                                    {
                                        transform: [
                                            {
                                                translateX: anim.interpolate({
                                                    inputRange: [0, 1],
                                                    outputRange: [0, 80],
                                                })
                                            }
                                        ],
                                        opacity: anim.interpolate({
                                            inputRange: [0, 1],
                                            outputRange: [1, 0.5]
                                        })
                                    }
                                ]
                            }
                        />
                    );
                })
            }
        </View>
    );
}

const styles = StyleSheet.create({
    box: {
        width: '100%',
        height: '100%',
        justifyContent: 'center',
        alignItems: 'center',
    },
    list: {
        width: 100,
        height: 50,
        borderRadius: 15,
        marginBottom: 5
    },
    list1: {
        backgroundColor: 'orange'
    },
    list2: {
        backgroundColor: 'purple'
    },
    list3: {
        backgroundColor: 'pink'
    }
});
```

:::

::: details loop(循环动画)。无限旋转。

```jsx
// loop 循环动画
import React, { useEffect, useRef } from "react";
import { Animated, Text, View, StyleSheet, Easing } from "react-native";

export default () => {
    const fadeAnim = useRef(new Animated.Value(0)).current;
    const animationFunc = useRef(() => Animated.timing(fadeAnim, {
        toValue: 1,
        duration: 2000,
        easing: Easing.linear
    })).current;

    useEffect(() => {
        Animated.loop(animationFunc(), {
            useNativeDriver: true,
        }).start()
    }, [])
    return <View style={styles.container}>
        <Animated.View
            style={[
                styles.fadingContainer,
                {
                    transform: [{
                        rotateZ: fadeAnim.interpolate({
                            inputRange: [0, 1],
                            outputRange: ['0deg', '360deg']
                        })
                    }]
                }
            ]}
        >
            <Text style={styles.fadingText}>😄</Text>
        </Animated.View>
    </View>
}

const styles = StyleSheet.create({
    container: {
        width: '100%',
        height: '100%',
        alignItems: "center",
        justifyContent: "center"
    },
    fadingContainer: {
        paddingVertical: 8,
        paddingHorizontal: 16,
        backgroundColor: "powderblue"
    },
    fadingText: {
        fontSize: 28,
        textAlign: "center",
        margin: 10
    },
});
```

:::

::: details loop(循环动画)。提示点击动画

```jsx
import React, { useEffect, useRef } from 'react';
import { View, Text, Animated, StyleSheet, Easing } from 'react-native';

export default () => {
    const offset = useRef(new Animated.Value(0)).current;
    const startAnimated = useRef(() => {
        const animationSlider = Animated.sequence([
            Animated.timing(offset, {
                toValue: 1,
                duration: 500,
                delay: 0,
                easing: Easing.linear,
            }),
            Animated.timing(offset, {
                toValue: 0,
                duration: 500,
                delay: 0,
                easing: Easing.linear,
            }),
        ]);
        Animated.loop(animationSlider).start();
    }).current;
    useEffect(() => {
        startAnimated();
    }, []);

    return (
        <View style={styles.container}>
            <View style={styles.button}>
                <Text style={styles.text}>点我点我</Text>
            </View>
            <Animated.Image
                source={require('./assets/hand.png')}
                style={[
                    styles.image,
                    {
                        transform: [
                            {
                                translateY: offset.interpolate({
                                    inputRange: [0, 1],
                                    outputRange: [0, 20]
                                })
                            }
                        ],
                    }
                ]}>
            </Animated.Image>
        </View>

    );
}

const styles = StyleSheet.create({
    container: {
        width: '100%',
        height: '100%',
        alignItems: 'center',
        justifyContent: 'center',
    },
    image: {
        width: 50,
        height: 50,
    },
    button: {
        padding: 10,
        justifyContent: 'center',
        alignItems: 'center',
        borderWidth: 2,
        borderColor: 'powderblue',
        borderRadius: 10
    }
})
```

:::

::: details 手势跟踪动画。拖拽

```jsx
  import React, { useRef } from "react";
import { Animated, View, StyleSheet, PanResponder } from "react-native";

const App = () => {
    const pan = useRef(new Animated.ValueXY()).current;
    const panResponder = useRef(
        PanResponder.create({
            onMoveShouldSetPanResponder: () => true,
            onPanResponderMove: Animated.event([
                null,
                { dx: pan.x, dy: pan.y }
            ]),
            onPanResponderRelease: () => {
                Animated.spring(pan, { toValue: { x: 0, y: 0 } }).start();
            }
        })
    ).current;

    return (
        <View style={styles.container}>
            <Animated.View
                style={{
                    transform: [{ translateX: pan.x }, { translateY: pan.y }]
                }}
                {...panResponder.panHandlers}
            >
                <View style={styles.box} />
            </Animated.View>
        </View>
    );
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        alignItems: "center",
        justifyContent: "center"
    },
    box: {
        height: 150,
        width: 150,
        backgroundColor: "blue",
        borderRadius: 5
    }
});

export default App;
```

:::

## 1 动画的使用

### 使用规则

1. 使用基本的 Animated 组件，如 Animated.View Animated.Image Animated.Text
2. 使用 Animated.Value 设定一个或多个初始化值（透明度，位置等等）
3. 将初始化值绑定到动画目标的属性上（如 style）
4. 通过 Animated.timing 等函数设定动画参数
5. 通过 start 函数开启动画

### 基本概念

#### 动画值

Animated. Value 可以定义一个动画值。如果需要实现类似元素的坐标变化等需求，可使用 animated. ValueXY 进行初始化。

#### 动画组件

经过特殊处理后的组件，动画值绑定到属性上。rn 自带的有：

* Animated. View 
* Animated. Image 
* Animated. Text
* Animated. ScrollView

除了直接使用 rn 自带的组件，还可以将自定义的组件封装成动画组件。通过 Animated.createAnimatedComponent() 用来处理组件，使其可以用于动画。

::: tip

使用动画组件的优点：

1. 避免react重新渲染和重新调和的开销。
2. 还能在组件卸载时做一些清理工作，使得这些组件在使用时是安全的。

:::

## 2 常用的动画函数

### 强大的 interpolate

插值函数，允许输入范围映射到不同的输出范围。

```js
 // 例子
 style = {
     {
         opacity: this.state.fadeAnim,
         transform: [{
             translateY: this.state.fadeAnim.interpolate({
                 inputRange: [0, 1],
                 outputRange: [0, 150]
             }),
         }],
     }
 }
```

interpolate 为强大的插值运算函数，当动画数值被 setValue(0.5) 时，对于以上代码的输出 [0, 150], 会被映射到75。当然支持多区段映射。

interpolate 一般使用场景为一个 animated.value 被应用到多个动画上。只要在属性里面设置好映射的值，就可以用一个动画变量来控制他们了。

interpolate 所支持的功能

1. 支持自定义多区间段落，常用来定义静止区间。
2. 支持映射到字符串。（角度的变化，例如: '90deg'）
3. 支持渐变函数

### 无限循环动画 loop

无限循环一个指定的动画，从头到尾周而复始。

::: tip
如果此循环的子动画设置了useNativeDriver: true则不会阻塞 JS 线程的执行。
::: 

#### 如果不用 loop, 该如何实现一个 loop 函数呢？

思路：当一次动画完成之后，在回调函数中再次执行动画函数。

```js
  function loop() {
      const animatedValue = new Animated.Value(5);
      Animated.timing(animatedValue, {
          toValue: 10,
          easing: Easing.linear,
          duration: 1000,
          useNativeDriver: true
      }).start((finished) => {
          finished && loop();
      });
  }
```

## 3 动画流程控制

**Animated.delay** 接受一个延迟时间作为参数，在指定的延迟时间过后执行动画。

**Animated.sequence** 接受一系列**动画数组**为参数，并依次执行。

**Animated.parallel** 接受一系列**动画数组**为参数，同时开始执行。

**Animated.stagger** 接受一系列动画数组和一个延迟时间，按照序列，每隔一个延迟时间后执行下一个动画（其实就是插入了 delay 的 parrllel）。

## 4 手势跟随动画

通过 animated.event 进行手势的跟随，panResponder 手势识别系统。

## 5 优化动画性能

### 启用原生动画驱动

在动画开始之前将所有动画的内容发送到原生代码，从而使用原生代码在ui线程上执行动画，不阻塞。

::: warning

1. 只适用于一些和布局无关的属性，像 transform 和 opacity。不支持作用于 position 的坐标属性。应用会报错提示。所以，位移运动应该尽量通过 transform 来实现。

:::
