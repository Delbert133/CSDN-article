> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/2201_75480019/article/details/142091482?spm=1001.2014.3001.5502)

随着 Android 开发技术的不断进步，Jetpack 提供的 ViewModel 和 LiveData 成为现代应用中不可或缺的工具。这两个组件可以帮助开发者更高效地管理 UI 状态，同时提高代码的可读性和维护性。在本篇博客中，我们将深入探讨它们的使用场景、核心功能及实际开发中的应用，并结合多个实战案例，提供详尽的代码实现和运行效果。

### **1. 为什么选择 ViewModel 和 LiveData？**

在传统 Android 开发中，**`Activity`** 和 **`Fragment`** 通常被用来处理大量的业务逻辑和数据状态管理。然而，这种设计方式可能带来以下问题：

*   **代码耦合过高**：UI 和业务逻辑混合在一起，难以扩展和测试。
    
*   **生命周期问题**：屏幕旋转或配置更改会导致 Activity 重建，从而丢失数据。
    
*   **异步更新复杂**：手动处理线程问题，容易出错。
    

Jetpack 的 ViewModel 和 LiveData 旨在解决这些问题：

#### **1.1 ViewModel 的核心功能**

*   **数据持久性**：即使在配置更改（如屏幕旋转）时，ViewModel 中的数据也不会丢失。
    
*   **与 UI 解耦**：ViewModel 专注于处理和管理数据，而不是直接操作 UI。
    

#### **1.2 LiveData 的核心功能**

*   **响应式编程**：LiveData 是一种可观察的数据持有类，UI 可以订阅 LiveData 的变化，从而实现自动更新。
    
*   **生命周期感知**：LiveData 会根据组件的生命周期自动管理订阅者，避免内存泄漏。
    

### **2. ViewModel 和 LiveData 的基本用法**

接下来，我们通过一个简单的计数器应用，展示如何使用 ViewModel 和 LiveData 实现高效的状态管理。

#### **2.1 创建项目并设置依赖**

首先，创建一个新的 Android 项目，并确保添加了必要的依赖。

在 **`build.gradle`**文件中添加以下依赖：

```
dependencies {
    implementation "androidx.lifecycle:lifecycle-viewmodel:2.5.1"
    implementation "androidx.lifecycle:lifecycle-livedata:2.5.1"
}

```

#### **2.2 创建 ViewModel**

接下来，定义一个**`CounterViewModel`** 类来管理计数器数据。

```
import androidx.lifecycle.LiveData;
import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;

public class CounterViewModel extends ViewModel {
    private final MutableLiveData<Integer> counter = new MutableLiveData<>(0);

    public LiveData<Integer> getCounter() {
        return counter;
    }

    public void incrementCounter() {
        if (counter.getValue() != null) {
            counter.setValue(counter.getValue() + 1);
        }
    }
}

```

#### **2.3 在 Activity 中使用 ViewModel 和 LiveData**

在 **`MainActivity`** 中，绑定 ViewModel 并观察 LiveData 的变化。

```
import android.os.Bundle;
import android.widget.Button;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import androidx.lifecycle.ViewModelProvider;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView counterTextView = findViewById(R.id.counterTextView);
        Button incrementButton = findViewById(R.id.incrementButton);

        // 创建 ViewModel 实例
        CounterViewModel counterViewModel = new ViewModelProvider(this).get(CounterViewModel.class);

        // 观察 LiveData 的变化
        counterViewModel.getCounter().observe(this, counter -> {
            counterTextView.setText(String.valueOf(counter));
        });

        // 设置按钮点击事件
        incrementButton.setOnClickListener(v -> counterViewModel.incrementCounter());
    }
}

```

#### **2.4 定义布局文件**

在 **`res/layout/activity_main.xml`** 中定义 UI。

```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">

    <TextView
        android:id="@+id/counterTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="0"
        android:textSize="32sp" />

    <Button
        android:id="@+id/incrementButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Increment" />

</LinearLayout>

```

运行项目后，点击按钮，计数值会实时更新。通过 ViewModel 的使用，即使屏幕旋转，计数器的状态也会被保存。

#### **2.5 动画效果演示**

录制屏幕操作并生成 GIF 动图展示：

![](https://i-blog.csdnimg.cn/direct/dd34490266d44c64a0d7144fade6a84f.gif)

### **3. 高级用法：组合 LiveData 和 Transformations**

在实际开发中，我们经常需要对数据进行转换或组合。Jetpack 提供了 **`Transformations`** 工具来处理这些场景。

#### **3.1 使用 Transformations.map 创建项目和 ViewModel**

新建一个项目，并创建 **`TemperatureViewModel`** 类，用于管理温度数据。

```
import androidx.lifecycle.LiveData;
import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.Transformations;
import androidx.lifecycle.ViewModel;

public class TemperatureViewModel extends ViewModel {
    private final MutableLiveData<Double> temperatureCelsius = new MutableLiveData<>(0.0);

    public LiveData<Double> getTemperatureCelsius() {
        return temperatureCelsius;
    }

    public LiveData<Double> getTemperatureFahrenheit() {
        return Transformations.map(temperatureCelsius, celsius -> celsius * 9/5 + 32);
    }

    public void setTemperatureCelsius(double celsius) {
        temperatureCelsius.setValue(celsius);
    }
}

```

#### **3.2 在 Activity 中使用 ViewModel**

在 Activity 中绑定 **`TemperatureViewModel`**，并观察温度变化。

```
import android.os.Bundle;
import android.widget.EditText;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;
import androidx.lifecycle.ViewModelProvider;

public class TemperatureActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_temperature);

        EditText inputCelsius = findViewById(R.id.inputCelsius);
        TextView outputFahrenheit = findViewById(R.id.outputFahrenheit);

        TemperatureViewModel viewModel = new ViewModelProvider(this).get(TemperatureViewModel.class);

        viewModel.getTemperatureFahrenheit().observe(this, fahrenheit -> {
            outputFahrenheit.setText(String.format("%.2f °F", fahrenheit));
        });

        inputCelsius.setOnEditorActionListener((v, actionId, event) -> {
            double celsius = Double.parseDouble(inputCelsius.getText().toString());
            viewModel.setTemperatureCelsius(celsius);
            return true;
        });
    }
}

```

#### **3.3 定义布局文件**

在 **`res/layout/activity_temperature.xml`** 中定义 UI。

```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">

    <EditText
        android:id="@+id/inputCelsius"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="Enter Celsius"
        android:inputType="numberDecimal" />

    <TextView
        android:id="@+id/outputFahrenheit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="24sp" />

</LinearLayout>

```

运行程序，输入摄氏温度，界面会自动更新显示对应的华氏温度。

#### **3.4 动画效果演示：**

录制屏幕操作并生成 GIF 动图展示：

![](https://i-blog.csdnimg.cn/direct/47fd0eb5cf8b4803a8481f7192af4278.gif)

### **4. ViewModel 和 LiveData 的最佳实践**

1.  **避免在 ViewModel 中引用 Context**：
    
    *   ViewModel 的生命周期比 Activity 长，直接引用 Context 可能导致内存泄漏。
        
    *   如果必须使用 Context，可以使用 `AndroidViewModel`。
        
2.  **保持 ViewModel 的职责单一**：
    
    *   ViewModel 应该只关注数据的处理，不应该处理 UI 逻辑。
        
3.  **使用 MediatorLiveData 组合多个 LiveData**：
    
    *   MediatorLiveData 可观察多个数据源并在其中之一发生变化时触发更新。
        
    
    ```
    private MediatorLiveData<String> combinedData = new MediatorLiveData<>();
    
    public LiveData<String> getCombinedData() {
        return combinedData;
    }
    
    public MyViewModel() {
        combinedData.addSource(liveData1, value -> updateCombinedData());
        combinedData.addSource(liveData2, value -> updateCombinedData());
    }
    
    private void updateCombinedData() {
        combinedData.setValue(liveData1.getValue() + " - " + liveData2.getValue());
    }
    
    ```
    
4.  **结合 Repository 模式管理数据源**：
    
    *   将所有数据操作（如网络请求、数据库操作）封装到 Repository 中，通过 ViewModel 调用，进一步解耦代码。
        
5.  **分离 UI 和业务逻辑**：
    
    *   避免在 ViewModel 中直接操作 UI，例如 Toast 或 Snackbar，这些应该通过事件或其他方式通知 UI 层。
        

### **5. 总结**

Jetpack 的 ViewModel 和 LiveData 为 Android 开发提供了强大的状态管理和数据观察能力。通过它们，开发者可以更高效地构建响应式应用，减少生命周期相关问题，提高代码的可维护性。

希望通过本篇博客，你对 ViewModel 和 LiveData 有了更深入的了解。如果你有任何问题或建议，欢迎在评论区分享！

作者：戴立斌

![](https://i-blog.csdnimg.cn/direct/c90a4a4f1cee4bbd92f673926611e5d4.gif)