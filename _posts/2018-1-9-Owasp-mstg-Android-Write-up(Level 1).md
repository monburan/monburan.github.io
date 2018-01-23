# 0x01

安装apk后打开，会检测设备是否已经 Root ，点击 OK 后程序退出。

# 0x02

先用dex2jar把apk转成jar文件

    d2j-dex2jar -o UnCrackable-Level1.jar UnCrackable-Level1.apk

然后用jadx-gui打开jar包查看代码，通过Root提示找到下面的函数：

    protected void onCreate(Bundle bundle) {
        if (c.a() || c.b() || c.c()) {
            a("Root detected!");
        }
        if (b.a(getApplicationContext())) {
            a("App is debuggable!");
        }
        super.onCreate(bundle);
        setContentView(2130903040);
    }

上面的函数调用了下面的代码来判断是否 Root：

    package sg.vantagepoint.a;

    import android.os.Build;
    import java.io.File;

    public class c {
        public static boolean a() {
            for (String file : System.getenv("PATH").split(":")) {
                if (new File(file, "su").exists()) {
                    return true;
                }
            }
            return false;
        }

        public static boolean b() {
            String str = Build.TAGS;
            return str != null && str.contains("test-keys");
        }

        public static boolean c() {
            for (String file : new String[]{"/system/app/Superuser.apk", "/system/xbin/daemonsu", "/system/etc/init.d/99SuperSUDaemon", "/system/bin/.ext/.su", "/system/etc/.has_su_daemon", "/system/etc/.installed_su_daemon", "/dev/com.koushikdutta.superuser.daemon/"}) {
                if (new File(file).exists()) {
                    return true;
                }
            }
            return false;
        }
    }

当满足上述的判断条件后，程序监听按钮，一旦按下就触发下述代码：

    package sg.vantagepoint.uncrackable1;

    import android.content.DialogInterface;
    import android.content.DialogInterface.OnClickListener;

    class b implements OnClickListener {
        final /* synthetic */ MainActivity a;

        b(MainActivity mainActivity) {
            this.a = mainActivity;
        }

        public void onClick(DialogInterface dialogInterface, int i) {
            System.exit(0);
        }
    }

如果使用 Frida 将上面的onClick覆盖，程序就不会退出。

    Java.perform(function() {
        console.log("[!] Starting Bypass RootCheck...");
        var Class_b = Java.use("sg.vantagepoint.uncrackable1.b");
        Class_b.onClick.overload('android.content.DialogInterface', 'int').implementation = function() {
            console.log("[*] Hacking onClick()...");
        }
        console.log("[!] Done...");
    });

成功绕过上述的 Root 检测。留在程序的主界面，只有输入框和按钮，尝试随便输入些字符，可以看到 “That's not it. Try again.” 继续尝试绕过这个检测。

在MainActivity中有如下代码：

    if (a.a(obj)) {
        create.setTitle("Success!");
        create.setMessage("This is the correct secret.");
    } else {
        create.setTitle("Nope...");
        create.setMessage("That's not it. Try again.");
    }

追踪 a.a 找到如下代码：

    public class a {
        public static boolean a(String str) {
            byte[] bArr = new byte[0];
            try {
                bArr = sg.vantagepoint.a.a.a(b("8d127684cbc37c17616d806cf50473cc"), Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0));
            } catch (Exception e) {
                Log.d("CodeCheck", "AES error:" + e.getMessage());
            }
            return str.equals(new String(bArr));
        }

上面的代码最后返回了两个字符串比较的结果，其中传入的字符串为输入的字符串，当对比结果为false 时，返回到刚刚的函数就会弹出 “That's not it. Try again.”。

通过 frida 尝试使 a() 返回 True：

    Java.perform(function() {
        console.log("[*] Show real input...")
        var Class_uncrackable1_a = Java.use("sg.vantagepoint.uncrackable1.a");
        Class_uncrackable1_a.a.overload('java.lang.String').implementation = function(arg1) {
            console.log("[!] Real input is: " + arg1);
            return true;
        };
    });

当然这不是最后结果，继续尝试分析代码，sg.vantagepoint.a.a.a() 的运行结果会与我们输入的字符串进行对比，所以继续追下这个函数。

    package sg.vantagepoint.a;

    import java.security.Key;
    import javax.crypto.Cipher;
    import javax.crypto.spec.SecretKeySpec;

    public class a {
        public static byte[] a(byte[] bArr, byte[] bArr2) {
            Key secretKeySpec = new SecretKeySpec(bArr, "AES/ECB/PKCS7Padding");
            Cipher instance = Cipher.getInstance("AES");
            instance.init(2, secretKeySpec);
            return instance.doFinal(bArr2);
        }
    }

只要获取到上面的函数的返回值，就能拿到bArr的值了。

    Java.perform(function() {
        console.log("[*] Decrypting Password...")
        var Class_aa = Java.use("sg.vantagepoint.a.a");
        Class_aa.a.overload('[B', '[B').implementation = function(arg1, arg2) {
            var result = this.a(arg1, arg2);
            console.log("[*] Show result: " + result);
        };
    });

查看下这个函数的返回值:

    # [*] Show result: 73,32,119,97,110,116,32,116,111,32,98,101,108,105,101,118,101

猜测是Unicode编码，转义下：

    Java.perform(function() {
        console.log("[*] Decrypting Password...")
        var Class_aa = Java.use("sg.vantagepoint.a.a");
        Class_aa.a.overload('[B', '[B').implementation = function(arg1, arg2) {
            var result = this.a(arg1, arg2);
            console.log("[*] Show result: " + result);
            var password = '';
            for (i = 0; i < result.length; i++) {
                password += String.fromCharCode(result[i]);
            };
            console.log("[!] Real Password is: " + password);
            return result;
        };
    });

最后输出：
    # [!] Real input is: 1234
    # [*] Show result: 73,32,119,97,110,116,32,116,111,32,98,101,108,105,101,118,101
    # [!] Real Password is: I want to believe