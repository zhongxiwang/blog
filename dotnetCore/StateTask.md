# 简单的状态机

```csharp
    public enum AnimationState
    {
        Walk = 1,
        Dead,
    }

    public abstract class State
    {
        abstract public int GetStateId { get; }
        abstract public void Enter(StateEvent data);
        abstract public void Execute(StateEvent data);
        abstract public void Exit(StateEvent data);
    }


    //状态运行时候的参数
    public class StateEvent
    {
        public string data;
    }

    //行走状态
    public class State_Walk : State
    {
        public const int ID = 1;

        public override int GetStateId
        {
            get
            {
                return ID;
            }
        }

        public override void Enter(StateEvent data)
        {
            Console.WriteLine("角色行走-进入");
        }

        public override void Execute(StateEvent data)
        {
            Console.WriteLine("角色行走-执行中");
        }

        public override void Exit(StateEvent data)
        {
            Console.WriteLine("角色行走-退出");
        }
    }

    //死亡状态
    public class State_Dead : State
    {
        public const int ID = 2;
        public override int GetStateId
        {
            get
            {
                return ID;
            }
        }

        public override void Enter(StateEvent data)
        {
            Console.WriteLine("角色死亡-进入");
        }

        public override void Execute(StateEvent data)
        {
            Console.WriteLine("角色死亡-执行中");
        }

        public override void Exit(StateEvent data)
        {
            Console.WriteLine("角色死亡-退出");
        }
    }

    public class StateMachine
    {
        private State currentState = null;
        private State previousState = null;
        private StateEvent dataEvent = null;
        private bool isStop;

        public State CurrentState
        {
            get
            {
                return currentState;
            }
        }
        public State PreviousState
        {
            get
            {
                return previousState;
            }
        }
        public bool IsStop
        {
            get
            {
                return isStop;
            }

            set
            {
                isStop = value;
            }
        }

        private State GetState(AnimationState animationState)
        {
            switch (animationState)
            {
                case AnimationState.Walk: return new State_Walk();
                case AnimationState.Dead: return new State_Dead();
            }
            return new State_Walk();
        }

        public void ChangeState(AnimationState animationState, StateEvent data, StateEvent previousData = null)
        {
            ChangeState(GetState(animationState), data, previousData);
        }

        public void ChangeState(State state, StateEvent data, StateEvent previousData = null)
        {
            //如果切换的状态就是本状态,就退出
            if (currentState != null && state.GetStateId == currentState.GetStateId)
                return;

            //退出上一个状态
            if (previousState != null)
                previousState.Exit(previousData);

            //设置进状态,进入新状态
            currentState = state;
            dataEvent = data;
            currentState.Enter(data);
        }




        public void Update()
        {
            if (currentState == null)
            {
                Console.WriteLine("当前没有状态可以执行");
                return;
            }
            else if (IsStop)
            {
                Console.WriteLine("状态机已经停止");
                return;
            }
            else
            {
                currentState.Execute(dataEvent);
            }
        }
    }



                StateMachine sm = new StateMachine();
            sm.ChangeState(AnimationState.Walk, new StateEvent() { data = "行走需要的参数" });
            sm.Update();       //执行行走状态 
            sm.Update();       //执行行走状态     

            sm.ChangeState(AnimationState.Dead, new StateEvent() { data = "死亡需要的参数" });
            sm.Update();
            sm.IsStop = true;   //停止状态机

            sm.Update();        //再次执行状态
```