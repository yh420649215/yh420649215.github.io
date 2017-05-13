单例模式的五种写法，懒汉式、饿汉式、双重锁检测、静态内部类模式。
- 懒汉式

`public class MySingleton {
	private static MySingleton mySingleton;
	private MySingleton() {
	}
	public static synchronized MySingleton getMySingleton() {
		if (mySingleton == null) {
			mySingleton = new MySingleton();
		}
		return mySingleton;
	}
}`
懒汉式是延迟加载的，优点在于资源利用率高，但第一次调用时的初始化工作会导致性能延迟，以后每次获取实例时也都要先判断实例是否被初始化，造成些许效率损失。并且迸发效率低，因为99%的情况下不需要同步。。
- 饿汉式

`public class MySingleton1 {
	private static MySingleton1 mySingleton1 = new MySingleton1();
	private MySingleton1() {
	};
	public static MySingleton1 getMySingleton() {
		return mySingleton1;
	}
}`
饿汉式因为在类创建的同时就实例化了静态对象，其资源已经初始化完成，所以第一次调用时更快，优势在于速度和反应时间，但是不管此单例会不会被使用，在程序运行期间会一直占据着一定的内存。
- 双重检测锁

`public class MySingleton2 {
	private MySingleton2() {
	};
	private static volatile  MySingleton2 mySingleton2 = null;
	public static MySingleton2 getMySingleton() {
		if (mySingleton2 == null) {
			synchronized (MySingleton2.class) {
				if (mySingleton2 == null) {
					mySingleton2 = new MySingleton2();
				}
			}
		}
		return mySingleton2;
	}
}`
双重检查锁定（DCL）方式也是延迟加载的，它唯一的问题是，由于 Java 编译器允许处理器乱序执行，在 JDK 版本小于 1.5 时会有 DCL 失效的问题（原因解释详见附录 2）。当然，现在大家使用的 JDK 普遍都已超过 1.4，只要在定义单例时加上 1.5 及以上版本具体化了的 volatile 关键字，即可保证执行的顺序，从而使单例起效。所以 DCL 方式是推荐的一种方式。
- 静态内部类模式

`public class MySingleton3 {
	private MySingleton3() {
	}
	private static class getSingleton {
		private static final MySingleton3 mySingleton3 = new MySingleton3();
	}
	public static MySingleton3 getMySingleton() {
		return getSingleton.mySingleton3;
	}
}`
这种方式利用了 classloder 的机制来保证初始化 instance 时只会有一个。需要注意的是：虽然它的名字中有“静态”两字，但它是属于“懒汉模式”的！！这种方式的 Singleton 类被装载时，只要 SingletonHolder 类还没有被主动使用，instance 就不会被初始化。只有在显式调用 getInstance() 方法时，才会装载 SingletonHolder 类，从而实例化对象。

“静态内部类”方式基本上弥补了 DCL 方式在 JDK 版本低于 1.5 时高并发环境失效的缺陷。《Java并发编程实践》中也指出 DCL 方式的“优化”是丑陋的，对静态内部类方式推崇备至。但是可能因为同大家创建单例时的思考习惯不太一致（根据单例模式的特点，一般首先想到的是通过 instance 判空来确保单例），此方式并不特别常见，然而它是所有懒加载的单例实现中适用范围最广、限制最小、最为推荐的一种。
- 枚举单例

`public enum  MySingleton5{
    SINGLETON;
}`
枚举不仅在创建实例的时候默认是线程安全的，而且在反序列化时可以自动防止重新创建新的对象。枚举类是在第一次访问时才被实例化，是懒加载的。它写法简单，并板上钉钉地保证了在任何情况（包括反序列化、反射、克隆）下都是一个单例。不过由于枚举是 JDK 1.5 才加入的特性，所以同 DCL 方式一样，它对 JDK 的版本也有要求。