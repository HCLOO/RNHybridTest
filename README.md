# RNHybridTest
Android Native+React Native混合开发入门实例

步骤（命令行使用cmd或Terminal）：
1.进入通过react-native init来初始化一个React Native项目：
  react-native init RNHybrid（格式：react-native init 项目名）
2.上述命令会初始化一个完成的名为RNHybrid的React Native项目，然后我们将RNHybrid目录里面的android目录删除，替换成我们已创建的Android项目
3.在Android项目中添加React Native所需要的依赖：
  app/build.gradle中添加：
    dependencies {
    ...
    compile "com.facebook.react:react-native:+" // From node_modules
    }
  project/build.gradle中添加：
    allprojects {
      repositories {
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
        ...
      }
      ...
    }
4.配置权限：
  在AndroidManifest.xml文件中添加权限：
    <uses-permission android:name="android.permission.INTERNET"/>
5.在RNHybrid目录下的index.js文件并添加如下代码：
  import { AppRegistry } from 'react-native';
  import App from './App';

  AppRegistry.registerComponent('App1', () => App);
6.在RNHybrid目录下的App.js文件并添加如下代码：
  import React, { Component } from "react";
  import { Image, FlatList, StyleSheet, Text, View } from "react-native";

  var REQUEST_URL =
    "https://raw.githubusercontent.com/facebook/react-native/0.51-stable/docs/MoviesExample.json";

  var styles = StyleSheet.create({
    container: {
      flex: 1,
      flexDirection: 'row',
      justifyContent: 'center',
      alignItems: 'center',
      backgroundColor: '#F5FCFF',
      marginBottom: 8
    },
    thumbnail: {
      width: 70,
      height: 110
    },
    rightContainer: {
      flex: 1
    },
    title: {
      fontSize: 16,
      marginBottom: 8,
      textAlign: 'center',
      color: '#000000'
    },
    year: {
      fontSize: 14,
      textAlign: 'center',
      color: '#555555'
    },
    list: {
      marginTop: 15,
      backgroundColor: '#F5FCFF'
    },
    text: {
      marginTop: 30,
      fontSize: 22,
      backgroundColor: '#F5FCFF',
      justifyContent: 'center',
      alignItems: 'center',
      textAlign: 'center',   
      justifyContent: 'center'
    }
  });

  export default class HelloWorldApp extends Component {
    constructor(props) {
      super(props);
      this.state = {
        data: [],
        loaded: false
      };
      // 在ES6中，如果在自定义的函数里使用了this关键字，则需要对其进行“绑定”操作，否则this的指向不对
      // 像下面这行代码一样，在constructor中使用bind是其中一种做法（还有一些其他做法，如使用箭头函数等）
      this.fetchData = this.fetchData.bind(this);
    }

    componentDidMount() {
      this.fetchData();
    }

    fetchData() {
      fetch(REQUEST_URL)
        .then((response) => response.json())
        .then((responseData) => {
          // 注意，这里使用了this关键字，为了保证this在调用时仍然指向当前组件，我们需要对其进行“绑定”操作
          this.setState({
            data: this.state.data.concat(responseData.movies),
            loaded: true
          });
        });
    }

    render() {
      if (!this.state.loaded) {
        return this.renderLoadingView();
      }

      return (
        <View> 
          <Text style={styles.text}>
            这是React Native界面
          </Text>
          <FlatList
          data={this.state.data}
          renderItem={this.renderMovie}
          style={styles.list}
          keyExtractor={item => item.id}
        />
        </View>
      );
    }

    renderLoadingView() {
      return (
        <View style={styles.container}>
          <Text>
            正在加载电影数据……
          </Text>
        </View>
      );
    }

    renderMovie({ item }) {
      // { item }是一种“解构”写法，请阅读ES2015语法的相关文档
      // item也是FlatList中固定的参数名，请阅读FlatList的相关文档
      return (
        <View style={styles.container}>
          <Image
            source={{uri: item.posters.thumbnail}}
            style={styles.thumbnail}
          />
          <View style={styles.rightContainer}>
            <Text style={styles.title}>{item.title}</Text>
            <Text style={styles.year}>{item.year}</Text>
          </View>
        </View>
      );
    }
  }
7.为React Native创建一个Activity来作为容器:
  创建RNPageActivity：
    public class RNPageActivity extends AppCompatActivity implements DefaultHardwareBackBtnHandler {
      private ReactRootView mReactRootView;
      private ReactInstanceManager mReactInstanceManager;
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);

          mReactRootView = new ReactRootView(this);
          mReactInstanceManager = ReactInstanceManager.builder()
              .setApplication(getApplication())
              .setBundleAssetName("index.android.bundle")
              .setJSMainModulePath("index")
              .addPackage(new MainReactPackage())
              .setUseDeveloperSupport(BuildConfig.DEBUG)
              .setInitialLifecycleState(LifecycleState.RESUMED)
              .build();
          // 这个"App1"名字一定要和我们在index.js中注册的名字保持一致AppRegistry.registerComponent()
          mReactRootView.startReactApplication(mReactInstanceManager, "App1", null);

          setContentView(mReactRootView);
      }

      @Override
      public void invokeDefaultOnBackPressed() {
          super.onBackPressed();
      }
  }
在中AndroidManifest.xml注册RNPageActivity：
  <activity
      android:name=".RNPageActivity"
      android:configChanges="keyboard|keyboardHidden|orientation|screenSize"
      android:windowSoftInputMode="adjustResize"
      android:theme="@style/Theme.AppCompat.Light.NoActionBar"/>
为ReactInstanceManager添加Activity的生命周期回调：
  @Override
  protected void onPause() {
      super.onPause();

      if (mReactInstanceManager != null) {
          mReactInstanceManager.onHostPause(this);
      }
  }

  @Override
  protected void onResume() {
      super.onResume();

      if (mReactInstanceManager != null) {
          mReactInstanceManager.onHostResume(this, this);
      }
  }

  @Override
  public void onBackPressed() {
      if (mReactInstanceManager != null) {
          mReactInstanceManager.onBackPressed();
      } else {
          super.onBackPressed();
      }
  }

  @Override
  protected void onDestroy() {
      super.onDestroy();

      if (mReactInstanceManager != null) {
          mReactInstanceManager.onHostDestroy(this);
      }
      if (mReactRootView != null) {
          mReactRootView.unmountReactApplication();
      }
  }
8.运行React Native，当发生红屏时，处理的方法有以下两种：
  （1）直接运行：
    首先确保你的电脑和手机设备在同一个 Wi-Fi 环境下。
    在设备上运行你的 React Native 应用。和打开其它 App 一样操作。
    你应该会看到一个“红屏”错误提示。这是正常的，下面的步骤会解决这个报错。
    摇晃设备，或者运行adb shell input keyevent 82，可以打开开发者菜单。
    点击Dev Settings -> Debug server host for device。
    输入你电脑的 IP 地址和端口号（譬如 10.0.1.1:8081）。在 Mac 上，你可以在系统设置/网络里找查询你的 IP 地址。在 Windows 上，打开命令提示符并输入ipconfig来查询你的 IP 地址。在 Linux 上你可以在终端中输入ifconfig来查询你的 IP 地址。
    回到开发者菜单然后选择Reload JS。
  （2）若（1）方法失效，可使用本方法：
    在Android项目的app/src/mian目录下创建assets目录。
    通过命令行切换到RNHybrid目录：
      若为旧版（即RNHybrid目录中存在index.android.js），则执行以下命令：
        react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output Android项目名/app/src/main/assets/index.android.bundle --assets-dest Android项目名/app/src/main/res
      若为新版（即RNHybrid目录中存在index.js），则执行以下命令：
        react-native bundle --platform android --dev false --entry-file index.js --bundle-output Android项目名/app/src/main/assets/index.android.bundle --assets-dest Android项目名/app/src/main/res
9.重新运行React Native即可。
