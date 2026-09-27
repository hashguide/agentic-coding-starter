---
kind: playbook
domain: experience
topics: [experience, interaction, optimistic, ui]
authority: guidance
---

# Optimistic UI

Optimistic UI updates the interface immediately when the user takes an action, before the server confirms success. The assumption is the action will succeed — in most cases, it does. When it fails, the UI rolls back to the previous state and surfaces an error.

## When to Use Optimistic UI

**Appropriate for:**
- High-frequency, low-risk interactions: likes, reactions, toggles, starring an item, reordering a list
- Actions where immediate feedback is meaningfully better than a loading state (the delay feels like a bug without optimism)
- Actions with low blast radius if they fail (rollback is clean and expected)

**Not appropriate for:**
- Payment or subscription changes — the user must know the charge succeeded before seeing confirmation
- Destructive actions (delete, hard remove) — optimistically removing something before server confirmation risks data loss if the delete fails and the rollback is missed
- State that other users will see — an optimistic update for one user that fails and rolls back while another user is watching creates inconsistency

## The Pattern

```tsx
import { useState } from 'react';

function LikeButton({ postId, initialLiked, initialCount }: LikeButtonProps) {
  const [liked, setLiked] = useState(initialLiked);
  const [count, setCount] = useState(initialCount);
  const [error, setError] = useState<string | null>(null);

  async function handleLike() {
    // 1. Capture previous state for rollback
    const prevLiked = liked;
    const prevCount = count;

    // 2. Optimistic update — change UI immediately
    setLiked(!prevLiked);
    setCount(prevLiked ? prevCount - 1 : prevCount + 1);
    setError(null);

    try {
      // 3. Send request in background
      await fetch(`/api/posts/${postId}/like`, {
        method: prevLiked ? 'DELETE' : 'POST',
      });
    } catch {
      // 4. Rollback on failure + surface error
      setLiked(prevLiked);
      setCount(prevCount);
      setError('Could not update. Try again.');
    }
  }

  return (
    <div>
      <button onClick={handleLike} aria-pressed={liked}>
        {liked ? '♥' : '♡'} {count}
      </button>
      {error && <span className="error-inline">{error}</span>}
    </div>
  );
}
```

## Rollback Handling

When a rollback occurs, the error must be visible near the element that was interacted with — not as a toast elsewhere on the screen. The user tapped a specific thing; the error belongs near that thing.

**Rollback checklist:**
- [ ] Previous state is captured before the optimistic update
- [ ] Rollback restores all optimistically modified state (not just the primary field)
- [ ] Error message is displayed inline near the affected element
- [ ] The user can retry the action from the error state

## With TanStack Query

TanStack Query's `useMutation` handles the optimistic update pattern with built-in rollback:

```tsx
const likeMutation = useMutation({
  mutationFn: (liked: boolean) =>
    fetch(`/api/posts/${postId}/like`, { method: liked ? 'POST' : 'DELETE' }),
  onMutate: async (newLiked) => {
    await queryClient.cancelQueries({ queryKey: ['post', postId] });
    const previous = queryClient.getQueryData(['post', postId]);
    queryClient.setQueryData(['post', postId], (old: Post) => ({
      ...old,
      liked: newLiked,
      likeCount: newLiked ? old.likeCount + 1 : old.likeCount - 1,
    }));
    return { previous };
  },
  onError: (_err, _newLiked, context) => {
    // Rollback
    queryClient.setQueryData(['post', postId], context?.previous);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['post', postId] });
  },
});
```

## Related Documents

- [Error feedback](error-feedback.md)
- [Frontend state management](../../frontend/state-management.md)
- [Micro-interactions](micro-interactions.md)
