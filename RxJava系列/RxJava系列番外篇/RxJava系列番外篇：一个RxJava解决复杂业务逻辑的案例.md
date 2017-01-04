# RxJava系列番外篇：一个RxJava解决复杂业务逻辑的案例

之前写过一系列RxJava1的文章，也承诺过会尽快有RxJava2的介绍。无奈实际项目中还未真正的使用RxJava2，不敢妄动笔墨。所以这次还是给大家分享一个使用RxJava1解决问题的案例，希望对大家在使用RxJava的时候有一点点启发。对RxJava还不了解的同学可以先去看看我之前的RxJava系列文章：

* [RxJava系列1(简介)](https://zhuanlan.zhihu.com/p/20687178)
* [RxJava系列2(基本概念及使用介绍)](https://zhuanlan.zhihu.com/p/20687307)
* [RxJava系列3(转换操作符)](https://zhuanlan.zhihu.com/p/21926591)
* [RxJava系列4(过滤操作符)](https://zhuanlan.zhihu.com/p/21966621)
* [RxJava系列5(组合操作符)](https://zhuanlan.zhihu.com/p/22039934)
* [RxJava系列6(从微观角度解读RxJava源码)](https://zhuanlan.zhihu.com/p/22338235)   
* [RxJava系列7(最佳实践)](https://zhuanlan.zhihu.com/p/23108381)  

## 业务场景

拿[MinimalistWeather](https://github.com/BaronZ88/MinimalistWeather)这个开源的天气App来举例：

进入App首页后，首先我们需要从数据库中获取当前城市的天气数据，如果数据库中存在天气数据则在UI页面上展示天气数据；如果数据库中未存储当前城市的天气数据，或者已存储的天气数据的发布时间相比现在已经超过了一小时，并且网络属于连接状态则调用API从服务端获取天气数据。如果获取到到的天气数据发布时间和当前数据库中的天气数据发布时间一致则丢弃掉从服务端获取到的天气数据，如果不一致则更新数据库并且在页面上展示最新的天气信息。（同时天气数据源是可配置的，可选择是小米天气数据源还是Know天气数据源）

## 解决方案

首先我们需要创建一个从数据库获取天气数据的Observable `observableForGetWeatherFromDB`，同时我们也需要创建一个从API获取天气数据的Observable `observableForGetWeatherFromNetWork`；为了在无网络状态下免于创建`observableForGetWeatherFromNetWork`我们在这之前需要首先判断下网络状态。最后使用`contact`操作符将两个Observable合并，同时使用`distinct`和`takeUntil`操作符来过滤筛选数据以符合业务需求，然后结合`subscribeOn`和`observeOn`做线程切换。上述这一套复杂的业务逻辑如果使用传统编码方式将是极其复杂的。下面我们来看看使用RxJava如何清晰简洁的来实现这个复杂的业务：

```Java
Observable<Weather> observableForGetWeatherData;
//首先创建一个从数据库获取天气数据的Observable
Observable<Weather> observableForGetWeatherFromDB = Observable.create(new Observable.OnSubscribe<Weather>() {
    @Override
    public void call(Subscriber<? super Weather> subscriber) {
        try {
            Weather weather = weatherDao.queryWeather(cityId);
            subscriber.onNext(weather);
            subscriber.onCompleted();
        } catch (SQLException e) {
            throw Exceptions.propagate(e);
        }
    }
});

if (!NetworkUtils.isNetworkConnected(context)) {
    observableForGetWeatherData = observableForGetWeatherFromDB;
} else {
    //接着创建一个从网络获取天气数据的Observable
    Observable<Weather> observableForGetWeatherFromNetWork = null;
    switch (configuration.getDataSourceType()) {
        case ApiConstants.WEATHER_DATA_SOURCE_TYPE_KNOW:
            observableForGetWeatherFromNetWork = ApiClient.weatherService.getKnowWeather(cityId)
                    .map(new Func1<KnowWeather, Weather>() {
                        @Override
                        public Weather call(KnowWeather knowWeather) {
                            return new KnowWeatherAdapter(knowWeather).getWeather();
                        }
                    });
            break;
        case ApiConstants.WEATHER_DATA_SOURCE_TYPE_MI:
            observableForGetWeatherFromNetWork = ApiClient.weatherService.getMiWeather(cityId)
                    .map(new Func1<MiWeather, Weather>() {
                        @Override
                        public Weather call(MiWeather miWeather) {
                            return new MiWeatherAdapter(miWeather).getWeather();
                        }
                    });
            break;
    }
    assert observableForGetWeatherFromNetWork != null;
    observableForGetWeatherFromNetWork = observableForGetWeatherFromNetWork
            .doOnNext(new Action1<Weather>() {
                @Override
                public void call(Weather weather) {
                    Schedulers.io().createWorker().schedule(() -> {
                        try {
                            weatherDao.insertOrUpdateWeather(weather);
                        } catch (SQLException e) {
                            throw Exceptions.propagate(e);
                        }
                    });
                }
            });

    //使用concat操作符将两个Observable合并
    observableForGetWeatherData = Observable.concat(observableForGetWeatherFromDB, observableForGetWeatherFromNetWork)
            .filter(new Func1<Weather, Boolean>() {
                @Override
                public Boolean call(Weather weather) {
                    return weather != null && !TextUtils.isEmpty(weather.getCityId());
                }
            })
            .distinct(new Func1<Weather, Long>() {
                @Override
                public Long call(Weather weather) {
                    return weather.getRealTime().getTime();//如果天气数据发布时间一致，我们再认为是相同的数据从丢弃掉
                }
            })
            .takeUntil(new Func1<Weather, Boolean>() {
                @Override
                public Boolean call(Weather weather) {
                    return System.currentTimeMillis() - weather.getRealTime().getTime() <= 60 * 60 * 1000;//如果天气数据发布的时间和当前时间差在一小时以内则终止事件流
                }
            });
}

observableForGetWeatherData.subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Action1<Weather>() {
            @Override
            public void call(Weather weather) {
                displayWeatherInformation();
            }
        }, new Action1<Throwable>() {
            @Override
            public void call(Throwable throwable) {
                Toast.makeText(context, throwable.getMessage(), Toast.LENGTH_LONG).show();
            }
        });
```

上面的代码看起来比较复杂，我们采用Lambda表达式简化下代码：

```Java
Observable<Weather> observableForGetWeatherData;
//首先创建一个从数据库获取天气数据的Observable
Observable<Weather> observableForGetWeatherFromDB = Observable.create(new Observable.OnSubscribe<Weather>() {
    @Override
    public void call(Subscriber<? super Weather> subscriber) {
        try {
            Weather weather = weatherDao.queryWeather(cityId);
            subscriber.onNext(weather);
            subscriber.onCompleted();
        } catch (SQLException e) {
            throw Exceptions.propagate(e);
        }
    }
});

if (!NetworkUtils.isNetworkConnected(context)) {
    observableForGetWeatherData = observableForGetWeatherFromDB;
} else {
    //接着创建一个从网络获取天气数据的Observable
    Observable<Weather> observableForGetWeatherFromNetWork = null;
    switch (configuration.getDataSourceType()) {
        case ApiConstants.WEATHER_DATA_SOURCE_TYPE_KNOW:
            observableForGetWeatherFromNetWork = ApiClient.weatherService.getKnowWeather(cityId)
                    .map(knowWeather -> new KnowWeatherAdapter(knowWeather).getWeather());
            break;
        case ApiConstants.WEATHER_DATA_SOURCE_TYPE_MI:
            observableForGetWeatherFromNetWork = ApiClient.weatherService.getMiWeather(cityId)
                    .map(miWeather -> new MiWeatherAdapter(miWeather).getWeather());
            break;
    }
    assert observableForGetWeatherFromNetWork != null;
    observableForGetWeatherFromNetWork = observableForGetWeatherFromNetWork
            .doOnNext(weather -> Schedulers.io().createWorker().schedule(() -> {
                try {
                    weatherDao.insertOrUpdateWeather(weather);
                } catch (SQLException e) {
                    throw Exceptions.propagate(e);
                }
            }));

    //使用concat操作符将两个Observable合并
    observableForGetWeatherData = Observable.concat(observableForGetWeatherFromDB, observableForGetWeatherFromNetWork)
            .filter(weather -> weather != null && !TextUtils.isEmpty(weather.getCityId()))
            .distinct(weather -> weather.getRealTime().getTime())//如果天气数据发布时间一致，我们再认为是相同的数据从丢弃掉
            .takeUntil(weather -> System.currentTimeMillis() - weather.getRealTime().getTime() <= 60 * 60 * 1000);//如果天气数据发布的时间和当前时间差在一小时以内则终止事件流
}

observableForGetWeatherData.subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(weather -> displayWeatherInformation(),
                throwable -> Toast.makeText(context, throwable.getMessage(), Toast.LENGTH_LONG).show());
```

## 小技巧

在上述的实现中有几点是我们需要注意的:

1. 为什么我需要在判断网络那块整个if else？这样看起来很不优雅，我们通过RxJava符完全可以实现同样的操作啊！之所以这样做是为了在无网络状况下去创建不必要的Observable `observableForGetWeatherFromNetWork`;

2. 更新数据库的操作不应该阻塞更新UI，因此我们在`observableForGetWeatherFromNetWork`的`doOnNext`中需要通过`Schedulers.io().createWorker()`去另起一条线程，以此保证更新数据库不会阻塞更新UI的操作。

	> 有同学可能会问为什么不在`doOnNext`之后再调用一次`observeOn`把更新数据库的操作切换到一条新的子线程去操作呢？其实一开始我也是这样做的，后来想想不对。整个Observable的事件传递处理就像是在一条流水线上完成的，虽然我们可以通过`observeOn`来指定子线程去处理更新数据库的操作，但是只有等这条子线程完成了更新数据库的任务后事件才会继续往后传递，这样就阻塞了更新UI的操作。对此有疑问的同学可以去看看我之前关于RxJava源码分析的文章或者自己动手debug看看。

## 问题

最后给大家留个两个问题：

1. 上述代码是最佳实现方案吗？还有什么更加合理的做法？
2. 我们在`observableForGetWeatherData`中使用`distinct`和`takeUntil`过滤筛选天气数据的时候网络请求会不会已经发出去了？这样做还有意义吗？

欢迎大家在留言中一起讨论。

> 本文中的代码在[MinimalistWeather](https://github.com/BaronZ88/MinimalistWeather)中的`WeatherDataRepository`类中有同样的实现，文章中为了更完整的将整个实现过程呈现出来，对代码做了部分改动。
> 
> 如果大家喜欢这一系列的文章，欢迎关注我的知乎专栏、Github以及简书。
>   
> * 知乎专栏：[https://zhuanlan.zhihu.com/baron](https://zhuanlan.zhihu.com/baron)  
> * GitHub：[https://github.com/BaronZ88](https://github.com/BaronZ88)
> * 简书：[http://www.jianshu.com/users/cfdc52ea3399](http://www.jianshu.com/users/cfdc52ea3399) 


