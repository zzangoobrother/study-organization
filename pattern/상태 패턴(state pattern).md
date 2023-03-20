# 상태 패턴(state pattern)
객체 내부 상태에 따라 스스로 행동을 변경할 수 있게 허가하는 패턴<br>
상태 전이를 위한 조건 로직이 지나치게 복잡한 경우 이를 해소하는 것이다.

````java
public abstract class State {
    protected Player player;

    public State(Player player) {
        this.player = player;
    }

    public abstract void standUp();
    public abstract void sitDown();
    public abstract void walk();
    public abstract void run();
    public abstract String getDescription();
}
````

````java
public class StandUpState extends State {
    public StandUpState(Player player) {
        super(player);
    }

    @Override
    public void standUp() {
        player.talk("언제 움직임?");
    }

    @Override
    public void sitDown() {
        player.setState(new SitDownState(player));
        player.talk("앉으니깐 좋다.");
    }

    @Override
    public void walk() {
        player.setSpeed(5);
        player.setState(new WalkState(player));
        player.talk("걷기가 좋다.");
    }

    @Override
    public void run() {
        player.setSpeed(10);
        player.setState(new RunState(player));
        player.talk("갑자기 뛰네요.");
    }

    @Override
    public String getDescription() {
        return "제자리에 서 있다.";
    }
}
````

````java
public class SitDownState extends State {
    public SitDownState(Player player) {
        super(player);
    }

    @Override
    public void standUp() {
        player.setState(new StandUpState(player));
        player.talk("일어나세요.");
    }

    @Override
    public void sitDown() {
        player.talk("나 앉아 있어.");
    }

    @Override
    public void walk() {
        player.talk("앉아서 어떻게 걸어? 일단 서자.");
        player.setState(new StandUpState(player));
    }

    @Override
    public void run() {
        player.talk("앉아서 어떻게 뛰어? 일단 서자.");
        player.setState(new StandUpState(player));
    }

    @Override
    public String getDescription() {
        return "앉아 있음";
    }
}
````

````java
public class WalkState extends State {
    public WalkState(Player player) {
        super(player);
    }

    @Override
    public void standUp() {
        player.setSpeed(0);
        player.talk("멈추세요!!");
        player.setState(new StandUpState(player));
    }

    @Override
    public void sitDown() {
        player.setSpeed(0);
        player.talk("걷다가 앉으면 넘어져요!");
        player.setState(new SitDownState(player));
    }

    @Override
    public void walk() {
        player.talk("걷는게 좋아요!");
    }

    @Override
    public void run() {
        player.setSpeed(20);
        player.talk("걷다가 뛰면 빠르지~!");
        player.setState(new RunState(player));
    }

    @Override
    public String getDescription() {
        return "걷는중";
    }
}
````

````java
public class RunState extends State {
    public RunState(Player player) {
        super(player);
    }

    @Override
    public void standUp() {
        player.setSpeed(0);
        player.talk("멈추세요!!");
        player.setState(new StandUpState(player));
    }

    @Override
    public void sitDown() {
        player.setSpeed(0);
        player.talk("뛰다 앉으면 넘어져요!");
        player.setState(new SitDownState(player));
    }

    @Override
    public void walk() {
        player.setSpeed(8);
        player.talk("걷는게 좋아요!");
        player.setState(new WalkState(player));
    }

    @Override
    public void run() {
        player.setSpeed(player.getSpeed() + 2);
        player.talk("더 빨리 뛰지~");
    }

    @Override
    public String getDescription() {
        return "뛰는 중";
    }
}
````

````java
public class Player {
    private int speed;
    private State state = new StandUpState(this);

    public void talk(String msg) {
        System.out.println("플레이어 : \"" + msg + "\"");
    }

    public int getSpeed() {
        return speed;
    }

    public void setSpeed(int speed) {
        this.speed = speed;
    }

    public State getState() {
        return state;
    }

    public void setState(State state) {
        this.state = state;
    }
}
````

````java
public class Main {
    public static void main(String[] args) {
        Player player = new Player();

        System.out.println(player.getState().getDescription() + " / 속도 : " + player.getSpeed() + "km/h");

        player.getState().walk();
        System.out.println(player.getState().getDescription() + " / 속도 : " + player.getSpeed() + "km/h");

        player.getState().run();
        System.out.println(player.getState().getDescription() + " / 속도 : " + player.getSpeed() + "km/h");

        player.getState().run();
        System.out.println(player.getState().getDescription() + " / 속도 : " + player.getSpeed() + "km/h");

        player.getState().standUp();
        System.out.println(player.getState().getDescription() + " / 속도 : " + player.getSpeed() + "km/h");

        player.getState().sitDown();
        System.out.println(player.getState().getDescription() + " / 속도 : " + player.getSpeed() + "km/h");
    }
}
````

같이 보면 좋은 자료
https://johngrib.github.io/wiki/pattern/state/
