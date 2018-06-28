

### LevelDB源码阅读笔记

---



#### 内存对齐

* leveldb的内存分配模块Arena中，有一个返回内存对齐的内存，源码为

  ```c
  char* Arena::AllocateAligned(size_t bytes) {
  		const int align = sizeof(void*); //We'll align to pointer size
  		assert((align & (align - 1)) == 0); //Pointer size should be a power of 2
  		size_t current_mod = reinterpret_cast<uintptr_t>(alloc_ptr_) & (align - 1);
  		size_t slop = (current_mod == 0 ? 0 : align - current_mod);
  		size_t needed = bytes + slop;
  		char* result;
  		if (needed <= alloc_bytes_remaining_) {
  			result = alloc_ptr_ + slop;
  			alloc_ptr_ += needed;
  			alloc_bytes_remaining_ -= needed;
  		} else {
  			result = AllocateFallback(bytes);
  		}
  		assert((reinterpret_cast<uintptr_t>(result) & (align - 1)) == 0);
  		return result;
  	}
  ```

  解析：align为需要对齐的位数（这里假设为4字节），用原指针地址 & （4 -1）得到的模mod，如果mod为0则表示原指针已经4字节对齐，如果不为0，则需要移动（align - mod）来是原指针地址4字节对齐。（4字节对齐的地址16进制后2位必须是0x00，（4-1）为0x11，他们进行&操作后结果为0，如果不为0，则进行&操作后的结果是需要移动位数）。