title: 经典的括号匹配问题与最小栈实现
slug: parentheses-match-using-stack
date: 2016-09-11 19:21:02
tags: leetcode, algorithm, stack

## Valid Parentheses

我觉得我实现的这个栈应该是挺简单的了吧.. 回头查查看别人的更短的实现. 不要在意 pop 里的 free 为何被注释掉了, 为了胜率...

	typedef
	struct stack {
	    char            c;
	    struct stack   *next;
	} Stack;
	
	void create(Stack **ptop)
	{
	    *ptop = NULL;
	}
	
	void push(Stack **ptop, char c)
	{
	    Stack   *tmp;
	    
	    tmp = malloc(sizeof *tmp);
	    tmp->c = c;
	    tmp->next = *ptop;
	    
	    *ptop = tmp;
	}
	
	void pop(Stack **ptop, char *c)
	{
	    Stack   *tmp;
	    
	    if (! *ptop) return;
	    
	    tmp = *ptop;
	    *c = tmp->c;
	    *ptop = (*ptop)->next;
	    //free(tmp);
	}
	
	int empty(Stack **ptop)
	{
	   return *ptop == NULL; 
	}
	
	bool isValid(char* s) {
	    Stack   *ss;
	    char    c;
	    
	    create(&ss);
	    while (*s) {
	        switch (*s) {
	        case '(':
	        case '[':
	        case '{':
	            push(&ss, *s);
	            break;
	        case ')':
	        case ']':
	        case '}':
	            if (empty(&ss)) return 0;
	            pop(&ss, &c);
	            if (c + 1 != *s
	                && c + 2 != *s) return 0;
	            break;
	        default:
	            break;
	        }
	        ++s;
	    }
	    
	    if (!empty(&ss)) return 0;
	    return 1;
	}

## Min Stack

这题得吐槽一下 Leetcode 的系统, 为什么到了这题就只能用 C++, Java, Python 三种语言? 为什么不能用 C 语言了???

	typedef
	struct stack {
	    int            c;
	    struct stack   *next;
	} C_Stack;
	
	void c_create(C_Stack **ptop)
	{
	    *ptop = NULL;
	}
	
	void c_push(C_Stack **ptop, int c)
	{
	    C_Stack   *tmp;
	    
	    tmp = (C_Stack *)malloc(sizeof *tmp);
	    tmp->c = c;
	    tmp->next = *ptop;
	    
	    *ptop = tmp;
	}
	
	void c_pop(C_Stack **ptop, int *c)
	{
	    C_Stack   *tmp;
	    
	    if (! *ptop) return;
	    tmp = *ptop;
	    *c = tmp->c;
	    *ptop = (*ptop)->next;
	    //free(tmp);
	}
	
	int c_empty(C_Stack **ptop)
	{
	   return *ptop == NULL; 
	}
	
	class MinStack {
	public:
	
	    C_Stack   *stack;
	    int     min;
	    
	    /** initialize your data structure here. */
	    MinStack() {
	        c_create(&stack);
	        min = INT_MAX;
	    }
	    
	    void push(int x) {
	        c_push(&stack, x);
	        if (x < min) min = x;
	    }
	    
	    void pop() {
	        int tmp;
	        C_Stack *p;
	        
	        c_pop(&stack, &tmp);
	        if (tmp != min)
	            return;
	            
	        min = INT_MAX;
	        for (p = stack ; p ; p = p->next) {
	            if (p->c < min)
	                min = p->c;
	        }
	    }
	    
	    int top() {
	        int tmp;
	        c_pop(&stack, &tmp);
	        c_push(&stack, tmp);
	        return tmp;
	    }
	    
	    int getMin() {
	        return min;
	    }
	};
	
	/**
	 * Your MinStack object will be instantiated and called as such:
	 * MinStack obj = new MinStack();
	 * obj.push(x);
	 * obj.pop();
	 * int param_3 = obj.top();
	 * int param_4 = obj.getMin();
	 */
