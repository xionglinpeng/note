# HashCode

## 说明

`hashCode()`是`java.lang.Object`中的方法

```java
@HotSpotIntrinsicCandidate
public native int hashCode();
```

它是一个`native`方法。





```java
The @HotSpotIntrinsicCandidate annotation is specific to the HotSpot Virtual Machine. It indicates that an annotated method may be (but is not guaranteed to be) intrinsified by the HotSpot VM. A method is intrinsified if the HotSpot VM replaces the annotated method with hand-written assembly and/or hand-written compiler IR -- a compiler intrinsic -- to improve performance. The @HotSpotIntrinsicCandidate annotation is internal to the Java libraries and is therefore not supposed to have any relevance for application code. Maintainers of the Java libraries must consider the following when modifying methods annotated with @HotSpotIntrinsicCandidate.
When modifying a method annotated with @HotSpotIntrinsicCandidate, the corresponding intrinsic code in the HotSpot VM implementation must be updated to match the semantics of the annotated method.
For some annotated methods, the corresponding intrinsic may omit some low-level checks that would be performed as a matter of course if the intrinsic is implemented using Java bytecodes. This is because individual Java bytecodes implicitly check for exceptions like NullPointerException and ArrayStoreException. If such a method is replaced by an intrinsic coded in assembly language, any checks performed as a matter of normal bytecode operation must be performed before entry into the assembly code. These checks must be performed, as appropriate, on all arguments to the intrinsic, and on other values (if any) obtained by the intrinsic through those arguments. The checks may be deduced by inspecting the non-intrinsic Java code for the method, and determining exactly which exceptions may be thrown by the code, including undeclared implicit RuntimeExceptions. Therefore, depending on the data accesses performed by the intrinsic, the checks may include:
null checks on references
range checks on primitive values used as array indexes
other validity checks on primitive values (e.g., for divide-by-zero conditions)
store checks on reference values stored into arrays
array length checks on arrays indexed from within the intrinsic
reference casts (when formal parameters are Object or some other weak type)
Note that the receiver value (this) is passed as a extra argument to all non-static methods. If a non-static method is an intrinsic, the receiver value does not need a null check, but (as stated above) any values loaded by the intrinsic from object fields must also be checked. As a matter of clarity, it is better to make intrinisics be static methods, to make the dependency on this clear. Also, it is better to explicitly load all required values from object fields before entering the intrinsic code, and pass those values as explicit arguments. First, this may be necessary for null checks (or other checks). Second, if the intrinsic reloads the values from fields and operates on those without checks, race conditions may be able to introduce unchecked invalid values into the intrinsic. If the intrinsic needs to store a value back to an object field, that value should be returned explicitly from the intrinsic; if there are multiple return values, coders should consider buffering them in an array. Removing field access from intrinsics not only clarifies the interface with between the JVM and JDK; it also helps decouple the HotSpot and JDK implementations, since if JDK code before and after the intrinsic manages all field accesses, then intrinsics can be coded to be agnostic of object layouts.
Maintainers of the HotSpot VM must consider the following when modifying intrinsics.
When adding a new intrinsic, make sure that the corresponding method in the Java libraries is annotated with @HotSpotIntrinsicCandidate and that all possible call sequences that result in calling the intrinsic contain the checks omitted by the intrinsic (if any).
When modifying an existing intrinsic, the Java libraries must be updated to match the semantics of the intrinsic and to execute all checks omitted by the intrinsic (if any).
Persons not directly involved with maintaining the Java libraries or the HotSpot VM can safely ignore the fact that a method is annotated with @HotSpotIntrinsicCandidate. The HotSpot VM defines (internally) a list of intrinsics. Not all intrinsic are available on all platforms supported by the HotSpot VM. Furthermore, the availability of an intrinsic on a given platform depends on the configuration of the HotSpot VM (e.g., the set of VM flags enabled). Therefore, annotating a method with @HotSpotIntrinsicCandidate does not guarantee that the marked method is intrinsified by the HotSpot VM. If the CheckIntrinsics VM flag is enabled, the HotSpot VM checks (when loading a class) that (1) all methods of that class that are also on the VM's list of intrinsics are annotated with @HotSpotIntrinsicCandidate and that (2) for all methods of that class annotated with @HotSpotIntrinsicCandidate there is an intrinsic in the list.


@Target({ElementType.METHOD, ElementType.CONSTRUCTOR})
@Retention(RetentionPolicy.RUNTIME)
public @interface HotSpotIntrinsicCandidate {
}
```



## hashCode()特性

`hashCode()`的特性已在其注释上有详细描述，如下：

Returns a hash code value for the object. This method is supported for the benefit of hash tables such as those provided by java.util.HashMap.
The general contract of hashCode is:
Whenever it is invoked on the same object more than once during an execution of a Java application, the hashCode method must consistently return the same integer, provided no information used in equals comparisons on the object is modified. This integer need not remain consistent from one execution of an application to another execution of the same application.
If two objects are equal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce the same integer result.
It is not required that if two objects are unequal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.
As much as is reasonably practical, the hashCode method defined by class Object does return distinct integers for distinct objects. (The hashCode may or may not be implemented as some function of an object's memory address at some point in time.)

**总结：**

`hashCode()`可以返回对象的哈希码值。之所以支持这个方法是因为针对集合类型数据结构提供可尽可能分散的删列表，比如`java.util.HashMap`。

返回对象的哈希码值。支持这种方法是因为散列表的好处，比如`java.util.HashMap`提供的散列表。

hashCode的一般特性如下:

1. 新建对象是没有hashCode的，只有当调用`hashCode()`方法之后，hashCode会被存储在Mark Word上。
2. 一旦获取hashCode之后，只要没有修改在对象上的相等比较中使用的信息，那之后每次调用`hashCode()`方法返回的都是存储在Mark Word上的hashCode，即每次返回都是一致的。
3. 每次启动应用程序，同一个对象获取的hashCode并不要求必须一致。
4. 如果根据`equals(Object)`方法，两个对象相等，那么对这两个对象调用hashCode方法必须产生相同的整数结果。
5. 如果根据`equals(Object)`方法，两个对象是不相等的，那么调用这两个对象上的hashCode方法必须产生不同的整数结果，这并不是必需的。但是，程序员应该意识到，为不等的对象生成不同的整数结果可能会提高哈希表的性能。
6. class Object定义的hashCode方法确实为不同的对象返回不同的整数，这是相当实际的。(hashCode可能实现，也可能不实现为某个时间点上对象内存地址的某个函数。)

如果`hashCode()`方法重写，那么上述只有第一条特性一定会被满足，其它的根据具体的实现而定。

## Object::hashCode()实现



```c++
intptr_t ObjectSynchronizer::FastHashCode (Thread * Self, oop obj) {
// 如果启用偏向锁
if (UseBiasedLocking) {
// NOTE: many places throughout the JVM do not expect a safepoint
// to be taken here, in particular most operations on perm gen
// objects. However, we only ever bias Java instances and all of
// the call sites of identity_hash that might revoke biases have
// been checked to make sure they can handle a safepoint. The
// added check of the bias pattern is to avoid useless calls to
// thread-local storage.
// 如果对象处于“已偏向”状态
if (obj-&gt;mark()-&gt;has_bias_pattern()) {
// Box and unbox the raw reference just in case we cause a STW safepoint.
// 将obj对象包装成一个句柄-&gt;hobj
Handle hobj (Self, obj) ;
// Relaxing assertion for bug 6320749.
// 保证程序的执行条件
assert (Universe::verify_in_progress() ||
!SafepointSynchronize::is_at_safepoint(),
"biases should not be seen by VM thread here");
// 撤销偏向锁
BiasedLocking::revoke_and_rebias(hobj, false, JavaThread::current());
// 这里根据Handle类定义的无参函数对象，将obj再取出来
obj = hobj() ;
// 看看是不是确保成功撤销了
assert(!obj-&gt;mark()-&gt;has_bias_pattern(), "biases should be revoked by now");
}
}

// hashCode() is a heap mutator ...
// Relaxing assertion for bug 6320749.
/**
   * 1. 确保当前的执行路径不处在全局安全点上；
   * 2. 确保当前线程是个JavaThread
   * 3. 确保当前线程没有被block
   */
assert (Universe::verify_in_progress() ||
!SafepointSynchronize::is_at_safepoint(), "invariant") ;
assert (Universe::verify_in_progress() ||
Self-&gt;is_Java_thread() , "invariant") ;
assert (Universe::verify_in_progress() ||
((JavaThread *)Self)-&gt;thread_state() != _thread_blocked, "invariant") ;

ObjectMonitor* monitor = NULL;
markOop temp, test;
intptr_t hash;
// 读出一个稳定的mark，防止obj正处于锁膨胀状态，如果正在膨胀，就等它膨胀完,再读出来
markOop mark = ReadStableMark (obj);

// object should remain ineligible for biased locking
// 我擦，你不会还处于“已偏向”状态吧
assert (!mark-&gt;has_bias_pattern(), "invariant") ;

// 如果mark现在处于中立状态了-&gt;unlock 这时候mark的结构应该是 [hash|age|0|01]
if (mark-&gt;is_neutral()) {
// 看看mark中是不是有一个存在的hash值
hash = mark-&gt;hash();              // this is a normal header
// 我靠，有了，省事了，直接返回吧
if (hash) {                       // if it has hash, just return it
return hash;
}
// 没有，那我就根据hashCode的算法规则重新算一个出来吧
hash = get_next_hash(Self, obj);  // allocate a new hash code
// 把上面的hash结果merge到mark中去，看到我写的那个结构了吗，放到hash那个位置
// 得到一个临时的temp，为什么这么干，继续看下面
temp = mark-&gt;copy_set_hash(hash); // merge the hash code into header
// use (machine word version) atomic operation to install the hash
// 上面的E文注释写得已经很直白了，CAS安装hash值咯
test = (markOop) Atomic::cmpxchg_ptr(temp, obj-&gt;mark_addr(), mark);
if (test == mark) {
// 看来CAS操作成功了，返回hash
return hash;
}
// If atomic operation failed, we must inflate the header
// into heavy weight monitor. We could add more code here
// for fast path, but it does not worth the complexity.
// 妈的，CAS失败了，上面E文说了，我们要把对象头膨胀成重量级锁了！
// 看看重锁状态时，mark的结构吧- monitor_ptr|10
// Anyway, 看看现在你是不是已经是重锁状态了吧
} else if (mark-&gt;has_monitor()) {
// 好家伙，你已经膨胀成重锁了嘛
monitor = mark-&gt;monitor();
// 那么我们就从ObjectMonitor对象中将mark取出来看看吧
temp = monitor-&gt;header();
// 这时候mark应该是无锁中立状态了，结构看上面吧！
assert (temp-&gt;is_neutral(), "invariant") ;
// 完事OK，取出来返回吧！
hash = temp-&gt;hash();
if (hash) {
return hash;
}
// Skip to the following code to reduce code size
// 锁对象正处在轻量级锁的状态，并且锁的持有者还是当前线程呢！
} else if (Self-&gt;is_lock_owned((address)mark-&gt;locker())) {
// 直接从线程栈里把mark取出来吧
temp = mark-&gt;displaced_mark_helper(); // this is a lightweight monitor owned
// mark不是中立状态？你搞笑吧！
assert (temp-&gt;is_neutral(), "invariant") ;
// 取出来返回咯
hash = temp-&gt;hash();              // by current thread, check if the displaced
if (hash) {                       // header contains hash code
return hash;
}
// WARNING:
//   The displaced header is strictly immutable.
// It can NOT be changed in ANY cases. So we have
// to inflate the header into heavyweight monitor
// even the current thread owns the lock. The reason
// is the BasicLock (stack slot) will be asynchronously
// read by other threads during the inflate() function.
// Any change to stack may not propagate to other threads
// correctly.
}
// 苦逼地等待你膨胀成重锁了...
// Inflate the monitor to set hash code
monitor = ObjectSynchronizer::inflate(Self, obj);
// Load displaced header and check it has hash code
// 从重锁对象中load出对象头mark来，看看是否已经有了一个hash值了
mark = monitor-&gt;header();
// check 不解释了
assert (mark-&gt;is_neutral(), "invariant") ;
hash = mark-&gt;hash();
// hash值还是空的，well，我们还是算一个出来吧！
// 下面的逻辑跟上面的一段一致，哥就不用费口舌了...
if (hash == 0) {
hash = get_next_hash(Self, obj);
temp = mark-&gt;copy_set_hash(hash); // merge hash code into header
assert (temp-&gt;is_neutral(), "invariant") ;
test = (markOop) Atomic::cmpxchg_ptr(temp, monitor, mark);
if (test != mark) {
// The only update to the header in the monitor (outside GC)
// is install the hash code. If someone add new usage of
// displaced header, please update this code
hash = test-&gt;hash();
assert (test-&gt;is_neutral(), "invariant") ;
assert (hash != 0, "Trivial unexpected object/monitor header usage.");
}
}
// We finally get the hash
// 费了好大劲，终于拿到hash值了，鸡冻~
return hash;
}
```







```c++
intptr_t ObjectSynchronizer::FastHashCode(Thread* self, oop obj) {
  //如果启用偏向锁
  if (UseBiasedLocking) {
    // NOTE: many places throughout the JVM do not expect a safepoint
    // to be taken here. However, we only ever bias Java instances and all
    // of the call sites of identity_hash that might revoke biases have
    // been checked to make sure they can handle a safepoint. The
    // added check of the bias pattern is to avoid useless calls to
    // thread-local storage.
    //如果对象处于已偏向状态（has_bias_pattern()：判断是否是偏向模式）  
    if (obj->mark().has_bias_pattern()) {
      // Handle for oop obj in case of STW safepoint
      Handle hobj(self, obj);
      if (SafepointSynchronize::is_at_safepoint()) {
        BiasedLocking::revoke_at_safepoint(hobj);
      } else {
        BiasedLocking::revoke(hobj, self);
      }
      obj = hobj();
      assert(!obj->mark().has_bias_pattern(), "biases should be revoked by now");
    }
  }

  while (true) {
    ObjectMonitor* monitor = NULL;
    markWord temp, test;
    intptr_t hash;
    markWord mark = read_stable_mark(obj);

    // object should remain ineligible for biased locking
    assert(!mark.has_bias_pattern(), "invariant");
	//如果当前对象处于无锁状态
    if (mark.is_neutral()) {            // if this is a normal header
      //查看Mark Word中的hash值，如果有，则直接返回
      hash = mark.hash();
      if (hash != 0) {                  // if it has a hash, just return it
        return hash;
      }
      //获取一个新的hash值
      hash = get_next_hash(self, obj);  // get a new hash
      temp = mark.copy_set_hash(hash);  // merge the hash into header
                                        // try to install the hash
      //使用CAS将新的hash值放置到Mark Word中  
      test = obj->cas_set_mark(temp, mark);
      //CAS成功，直接返回它
      if (test == mark) {               // if the hash was installed, return it
        return hash;
      }
      // Failed to install the hash. It could be that another thread
      // installed the hash just before our attempt or inflation has
      // occurred or... so we fall thru to inflate the monitor for
      // stability and then install the hash.
    //如果当前对象处于重量级锁状态[]    
    } else if (mark.has_monitor()) {
      monitor = mark.monitor();
      temp = monitor->header();
      assert(temp.is_neutral(), "invariant: header=" INTPTR_FORMAT, temp.value());
      hash = temp.hash();
      if (hash != 0) {
        // It has a hash.

        // Separate load of dmw/header above from the loads in
        // is_being_async_deflated().

        // dmw/header and _contentions may get written by different threads.
        // Make sure to observe them in the same order when having several observers.
        OrderAccess::loadload_for_IRIW();

        if (monitor->is_being_async_deflated()) {
          // But we can't safely use the hash if we detect that async
          // deflation has occurred. So we attempt to restore the
          // header/dmw to the object's header so that we only retry
          // once if the deflater thread happens to be slow.
          monitor->install_displaced_markword_in_object(obj);
          continue;
        }
        return hash;
      }
      // Fall thru so we only have one place that installs the hash in
      // the ObjectMonitor.
    //如果当前对象处于轻量级锁状态
    } else if (self->is_lock_owned((address)mark.locker())) {
      // This is a stack lock owned by the calling thread so fetch the
      // displaced markWord from the BasicLock on the stack.
      //直接从displaced mark word中取出hash，如果有，则直接返回
      temp = mark.displaced_mark_helper();
      assert(temp.is_neutral(), "invariant: header=" INTPTR_FORMAT, temp.value());
      hash = temp.hash();
      if (hash != 0) {                  // if it has a hash, just return it
        return hash;
      }
      // WARNING:
      // The displaced header in the BasicLock on a thread's stack
      // is strictly immutable. It CANNOT be changed in ANY cases.
      // So we have to inflate the stack lock into an ObjectMonitor
      // even if the current thread owns the lock. The BasicLock on
      // a thread's stack can be asynchronously read by other threads
      // during an inflate() call so any change to that stack memory
      // may not propagate to other threads correctly.
    }

    // Inflate the monitor to set the hash.

    // An async deflation can race after the inflate() call and before we
    // can update the ObjectMonitor's header with the hash value below.
    //将其膨胀为重量级锁
    monitor = inflate(self, obj, inflate_cause_hash_code);
    // Load ObjectMonitor's header/dmw field and see if it has a hash.
    mark = monitor->header();
    assert(mark.is_neutral(), "invariant: header=" INTPTR_FORMAT, mark.value());
    hash = mark.hash();
    if (hash == 0) {                    // if it does not have a hash
      hash = get_next_hash(self, obj);  // get a new hash
      temp = mark.copy_set_hash(hash);  // merge the hash into header
      assert(temp.is_neutral(), "invariant: header=" INTPTR_FORMAT, temp.value());
      uintptr_t v = Atomic::cmpxchg((volatile uintptr_t*)monitor->header_addr(), mark.value(), temp.value());
      test = markWord(v);
      if (test != mark) {
        // The attempt to update the ObjectMonitor's header/dmw field
        // did not work. This can happen if another thread managed to
        // merge in the hash just before our cmpxchg().
        // If we add any new usages of the header/dmw field, this code
        // will need to be updated.
        hash = test.hash();
        assert(test.is_neutral(), "invariant: header=" INTPTR_FORMAT, test.value());
        assert(hash != 0, "should only have lost the race to a thread that set a non-zero hash");
      }
      if (monitor->is_being_async_deflated()) {
        // If we detect that async deflation has occurred, then we
        // attempt to restore the header/dmw to the object's header
        // so that we only retry once if the deflater thread happens
        // to be slow.
        monitor->install_displaced_markword_in_object(obj);
        continue;
      }
    }
    // We finally get the hash.
    return hash;
  }
}
```

*get_next_hash*

```java
// hashCode() generation :
//
// Possibilities:
// * MD5Digest of {obj,stw_random}
// * CRC32 of {obj,stw_random} or any linear-feedback shift register function.
// * A DES- or AES-style SBox[] mechanism
// * One of the Phi-based schemes, such as:
//   2654435761 = 2^32 * Phi (golden ratio)
//   HashCodeValue = ((uintptr_t(obj) >> 3) * 2654435761) ^ GVars.stw_random ;
// * A variation of Marsaglia's shift-xor RNG scheme.
// * (obj ^ stw_random) is appealing, but can result
//   in undesirable regularity in the hashCode values of adjacent objects
//   (objects allocated back-to-back, in particular).  This could potentially
//   result in hashtable collisions and reduced hashtable efficiency.
//   There are simple ways to "diffuse" the middle address bits over the
//   generated hashCode values:

static inline intptr_t get_next_hash(Thread* self, oop obj) {
  intptr_t value = 0;
  if (hashCode == 0) {
    // This form uses global Park-Miller RNG.
    // On MP system we'll have lots of RW access to a global, so the
    // mechanism induces lots of coherency traffic.
    value = os::random();
  } else if (hashCode == 1) {
    // This variation has the property of being stable (idempotent)
    // between STW operations.  This can be useful in some of the 1-0
    // synchronization schemes.
    intptr_t addr_bits = cast_from_oop<intptr_t>(obj) >> 3;
    value = addr_bits ^ (addr_bits >> 5) ^ GVars.stw_random;
  } else if (hashCode == 2) {
    value = 1;            // for sensitivity testing
  } else if (hashCode == 3) {
    value = ++GVars.hc_sequence;
  } else if (hashCode == 4) {
    value = cast_from_oop<intptr_t>(obj);
  } else {
    // Marsaglia's xor-shift scheme with thread-specific state
    // This is probably the best overall implementation -- we'll
    // likely make this the default in future releases.
    unsigned t = self->_hashStateX;
    t ^= (t << 11);
    self->_hashStateX = self->_hashStateY;
    self->_hashStateY = self->_hashStateZ;
    self->_hashStateZ = self->_hashStateW;
    unsigned v = self->_hashStateW;
    v = (v ^ (v >> 19)) ^ (t ^ (t >> 8));
    self->_hashStateW = v;
    value = v;
  }

  value &= markWord::hash_mask;
  if (value == 0) value = 0xBAD;
  assert(value != markWord::no_hash, "invariant");
  return value;
}
```

`get_next_hash()`是实际生成哈希码值的函数，可以看到它有6个`if`分支，变量`hashCode`是一个虚拟机参数，可以通过`-XX:hashCode=n`设置，其默认值为5，定义在[globals.hpp](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/runtime/globals.hpp)：

```c++
product(intx, hashCode, 5, EXPERIMENTAL, "(Unstable) select hashCode generation algorithm") 
```

需要注意的是`-XX:hashCode=n`是一个实验性质（experimental）的参数，因此在`-XX:hashCode=n`之前需要添加`-XX:+UnlockExperimentalVMOptions`用于解锁试验性质的参数。

```jvm
-XX:+UnlockExperimentalVMOptions 
-XX:hashCode=2
```

## `-XX:hashCode` 5分支

- `-XX:hashCode=0`

当`-XX:hashCode=0`时，是利用Park-Miller随机数生成器（RNG - Random Number Generator）生成的哈希值，调用的是`os::random()`方法，源码位于[os.cpp](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/runtime/os.cpp)。

相关源码：

```c++
......
volatile unsigned int os::_rand_seed      = 1234567;
......
int os::next_random(unsigned int rand_seed) {
  /* standard, well-known linear congruential random generator with
   * next_rand = (16807*seed) mod (2**31-1)
   * see
   * (1) "Random Number Generators: Good Ones Are Hard to Find",
   *      S.K. Park and K.W. Miller, Communications of the ACM 31:10 (Oct 1988),
   * (2) "Two Fast Implementations of the 'Minimal Standard' Random
   *     Number Generator", David G. Carta, Comm. ACM 33, 1 (Jan 1990), pp. 87-88.
  */
  const unsigned int a = 16807;
  const unsigned int m = 2147483647;
  const int q = m / a;        assert(q == 127773, "weird math");
  const int r = m % a;        assert(r == 2836, "weird math");

  // compute az=2^31p+q
  unsigned int lo = a * (rand_seed & 0xFFFF);
  unsigned int hi = a * (rand_seed >> 16);
  lo += (hi & 0x7FFF) << 16;

  // if q overflowed, ignore the overflow and increment q
  if (lo > m) {
    lo &= m;
    ++lo;
  }
  lo += hi >> 15;

  // if (p+q) overflowed, ignore the overflow and increment (p+q)
  if (lo > m) {
    lo &= m;
    ++lo;
  }
  return lo;
}

int os::random() {
  // Make updating the random seed thread safe.
  while (true) {
    unsigned int seed = _rand_seed;
    unsigned int rand = next_random(seed);
    if (Atomic::cmpxchg(&_rand_seed, seed, rand) == seed) {
      return static_cast<int>(rand);
    }
  }
}
```

### 1、`-XX:hashCode=1`

首先是获取`oop`的地址，然后将地址经过`addr_bits ^ (addr_bits >> 5) ^ GVars.stw_random`运算使其哈希值更加散列化，减少哈希碰撞。

获取oop地址的相关源码位于[oopsHierarchy.hpp](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/oops/oopsHierarchy.hpp)：

```c++
// For CHECK_UNHANDLED_OOPS, it is ambiguous C++ behavior to have the oop
// structure contain explicit user defined conversions of both numerical
// and pointer type. Define inline methods to provide the numerical conversions.
template <class T> inline oop cast_to_oop(T value) {
  return (oop)(CHECK_UNHANDLED_OOPS_ONLY((void *))(value));
}
template <class T> inline T cast_from_oop(oop o) {
  return (T)(CHECK_UNHANDLED_OOPS_ONLY((oopDesc*))o);
}
```

### 2、`-XX:hashCode=2`

当`-XX:hashCode=2`时，是用于敏感度测试，例如某些集合是否对哈希值敏感。固定返回1。

### 3、`-XX:hashCode=3`

当`-XX:hashCode=3`时，利用自增序列生成哈希值。

`hc_sequence`是一个全局的int型变量，当每次调用任何对象的`hashCode()`方法时候，其值加1。源码就位于[synchronizer.cpp](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/runtime/synchronizer.cpp)中。

```c++
struct SharedGlobals {
  char         _pad_prefix[OM_CACHE_LINE_SIZE];
  // This is a highly shared mostly-read variable.
  // To avoid false-sharing it needs to be the sole occupant of a cache line.
  volatile int stw_random;
  DEFINE_PAD_MINUS_SIZE(1, OM_CACHE_LINE_SIZE, sizeof(volatile int));
  // Hot RW variable -- Sequester to avoid false-sharing
  volatile int hc_sequence;
  DEFINE_PAD_MINUS_SIZE(2, OM_CACHE_LINE_SIZE, sizeof(volatile int));
};

static SharedGlobals GVars;
```

**验证**

```java
/**
  * -XX:+UnlockExperimentalVMOptions -XX:hashCode=3
  */
public static void main(String[] args) {
    System.out.println(new User().hashCode());
    System.out.println(new User().hashCode());
    System.out.println(new User().hashCode());
    System.out.println(new User().hashCode());
    System.out.println(new User().hashCode());
}
```

测试结果

```
917
918
919
920
921
```

验证代码中，将`-XX:hashCode`设置为3，测试输出的结果顺序递增。注意，这里之所以没有从1开始，是因为虚拟机启动本身就会加载很多的类，其中有些类调用了`hashCode()`方法。

### 4、`-XX:hashCode=4`

跟`-XX:hashCode=1`时差不多，只是当`-XX:hashCode=4`时，是直接以`oop`的地址作为哈希值。

### 5、`-XX:hashCode=5`

利用Marsaglia's xor-shift随机数生成算法生成哈希值。

xor-shift算法参考资料

维基百科：https://en.wikipedia.org/wiki/Xorshift

Xorshift RNGs（PDF）：http://www.jstatsoft.org/v08/i14/paper

PDF：https://www.jstatsoft.org/article/view/v008i14/xorshift.pdf（https://www.jstatsoft.org/article/view/v008i14）

## 参考资料

- [hashcode() 方法底层实现](https://zhanghaoxin.blog.csdn.net/article/details/108627063)

