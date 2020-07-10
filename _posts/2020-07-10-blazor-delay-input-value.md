---
layout: post
title: "Blazor - delay input value"
date: 2020-07-09 23:10:13
background: '/assets/img/clocks01.jpg'
---

In this post we're going to learn how to delay value entered into input field when using Blazor. Before we continue it is expected that you already know a little about Blazor as this is considered to be more advanced lesson.

## Example

You might wonder why you would want to delay value from being submitted. Well to show you, we're going to first create a new `TextField` component around `<input>` element and `@oninput` event.

So, open your Blazor project and create new file named `TextField.razor`. Paste the following code:

```cs
<input type="text" value="@Text" @oninput="@OnInputHandler" />

@code {

    async Task OnInputHandler( ChangeEventArgs e )
    {
        Text = e?.Value?.ToString();

        await TextChanged.InvokeAsync( Text );
    }

    [Parameter] public string Text { get; set; }

    [Parameter] public EventCallback<string> TextChanged { get; set; }
}
```

Go to one of your pages and use this piece of code:

```cs
<TextField @bind-Text="@value"></TextField>

<p>@value</p>

@code{
    string value = "This is long text just for an example";
}
```

Now, start the applications and start typing into the middle of the newly created field. Type real fast! At this moment you might or might not stumble upon some really weird problem. Text caret could jump to the end of the field every time you try to enter anything. If the problem doesn't happen to you then you're one lucky guy. Give yourself a pat on the shoulder!

But since you're here I would guess the problem indeed happens to you. And you're in luck, cause here comes the solution.

## Solution

Create new class `ValueDelayer` and copy the following code:

```cs
using System.Timers;

/// <summary>
/// Delays the entered value by the defined interval.
/// </summary>
public class ValueDelayer : IDisposable
{
    /// <summary>
    /// Internal timer used to delay the value.
    /// </summary>
    private Timer timer;

    /// <summary>
    /// Holds the last updated value.
    /// </summary>
    private string value;

    /// <summary>
    /// Event raised after the interval has passed and with new updated value.
    /// </summary>
    public event EventHandler<string> Delayed;

    /// <summary>
    /// Default constructor.
    /// </summary>
    /// <param name="interval">Interval by which the value will be delayed.</param>
    public ValueDelayer( int interval )
    {
        timer = new Timer( interval );
        timer.Elapsed += OnElapsed;
        timer.AutoReset = false;
    }

    private void OnElapsed( object source, ElapsedEventArgs e )
    {
        Delayed?.Invoke( this, value );
    }

    /// <summary>
    /// Updates the internal value.
    /// </summary>
    /// <param name="value">New value.</param>
    public void Update( string value )
    {
        timer.Stop();

        this.value = value;

        timer.Start();
    }

    /// <summary>
    /// Releases all subscribed events.
    /// </summary>
    public void Dispose()
    {
        if ( timer != null )
        {
            timer.Stop();
            timer.Elapsed -= OnElapsed;
            timer = null;
        }
    }
}
```

Modify `TextField.razor` according to this:

```cs
@inherits ComponentBase
@implements IDisposable

<input type="text" value="@Text" @oninput="@OnInputHandler" />

@code {
    private ValueDelayer inputValueDelayer;

    protected override void OnInitialized()
    {
        inputValueDelayer = new ValueDelayer( DelayInterval );
        inputValueDelayer.Delayed += OnInputValueDelayed;

        base.OnInitialized();
    }

    public void Dispose()
    {
        inputValueDelayer.Delayed -= OnInputValueDelayed;
        inputValueDelayer = null;
    }

    private void OnInputValueDelayed( object sender, string value )
    {
        InvokeAsync( async () =>
        {
            Text = value;

            await TextChanged.InvokeAsync( Text );
        } );
    }

    async Task OnInputHandler( ChangeEventArgs e )
    {
        inputValueDelayer.Update( e?.Value?.ToString() );

        await TextChanged.InvokeAsync( Text );
    }

    [Parameter] public string Text { get; set; }

    [Parameter] public EventCallback<string> TextChanged { get; set; }

    [Parameter] public int DelayInterval { get; set; } = 300;
}
```

Run the application and start typing again. Voila, it works!