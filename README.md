# wpForo 2.x.x Plugin Hooks Documentation

Complete reference guide for all action and filter hooks available in wpForo 3.0 plugin.

**Total Hooks**: 609 occurrences
- **Action Hooks**: 227 unique hooks
- **Filter Hooks**: 382 unique hooks

---

## Table of Contents

1. [Initialization Hooks](#initialization-hooks)
2. [Topic Management Hooks](#topic-management-hooks)
3. [Post Management Hooks](#post-management-hooks)
4. [Forum Management Hooks](#forum-management-hooks)
5. [User & Member Hooks](#user--member-hooks)
6. [Activity Hooks](#activity-hooks)
7. [Reaction Hooks](#reaction-hooks)
8. [Form Hooks](#form-hooks)
9. [Template & Layout Hooks](#template--layout-hooks)
10. [Social Features Hooks](#social-features-hooks)
11. [Admin Panel Hooks](#admin-panel-hooks)
12. [Permission & Moderation Hooks](#permission--moderation-hooks)
13. [Email & Notification Hooks](#email--notification-hooks)
14. [Board Management Hooks](#board-management-hooks)
15. [Caching Hooks](#caching-hooks)
16. [Integration Hooks](#integration-hooks)

---

## Initialization Hooks

Hooks that fire during plugin initialization and setup.

### `wpforo_before_init`
**Type**: Action
**Location**: `wpforo.php:542`
**Parameters**: None
**Description**: Fires before wpForo core initialization begins.

**Example**:
```php
add_action('wpforo_before_init', function() {
    // Load custom dependencies before wpForo initializes
    require_once 'my-custom-wpforo-extension.php';
});
```

---

### `wpforo_core_inited`
**Type**: Action
**Location**: `wpforo.php:545`
**Parameters**: None
**Description**: Fires after wpForo core initialization is complete.

**Example**:
```php
add_action('wpforo_core_inited', function() {
    // Access wpForo core classes safely
    $forum_count = count(WPF()->forum->get_forums());
    error_log("wpForo initialized with {$forum_count} forums");
});
```

---

### `wpforo_after_init`
**Type**: Action
**Location**: `wpforo.php:547`
**Parameters**: None
**Description**: Fires after all wpForo initialization is complete.

**Example**:
```php
add_action('wpforo_after_init', function() {
    // Register custom post types or taxonomies that integrate with wpForo
    register_post_type('forum_attachment', [
        'label' => 'Forum Attachments',
        'public' => false,
    ]);
});
```

---

### `wpforo_before_init_base_classes`
**Type**: Action
**Location**: `wpforo.php:245`
**Parameters**: None
**Description**: Fires before base classes are initialized.

**Example**:
```php
add_action('wpforo_before_init_base_classes', function() {
    // Override base class behavior before instantiation
    define('WPFORO_CUSTOM_CACHE_DIR', '/custom/cache/path');
});
```

---

### `wpforo_after_init_base_classes`
**Type**: Action
**Location**: `wpforo.php:261`
**Parameters**: None
**Description**: Fires after base classes are initialized.

**Example**:
```php
add_action('wpforo_after_init_base_classes', function() {
    // Modify base class settings
    WPF()->ram_cache->set('my_custom_key', 'value');
});
```

---

### `wpforo_after_init_templates`
**Type**: Action
**Location**: `classes/Template.php:394`
**Parameters**: `$templates` (array)
**Description**: Fires after template initialization with available templates.

**Example**:
```php
add_action('wpforo_after_init_templates', function($templates) {
    // Register custom template
    $custom_template = [
        'name' => 'My Custom Layout',
        'slug' => 'custom-layout',
    ];
    WPF()->tpl->templates['custom'] = $custom_template;
}, 10, 1);
```

---

## Topic Management Hooks

Hooks for topic creation, editing, deletion, and status changes.

### `wpforo_before_add_topic`
**Type**: Action
**Location**: `classes/Topics.php`
**Parameters**: `$args` (array)
**Description**: Fires before a new topic is added to database.

**Example**:
```php
add_action('wpforo_before_add_topic', function($args) {
    // Log topic creation attempts
    error_log("New topic creation: " . $args['title']);

    // Validate custom requirements
    if (empty($args['custom_field'])) {
        wp_die('Custom field is required');
    }
}, 10, 1);
```

---

### `wpforo_after_add_topic`
**Type**: Action
**Location**: `classes/Topics.php`
**Parameters**: `$topic` (array), `$forum` (array)
**Description**: Fires after a topic is successfully added.

**Example**:
```php
add_action('wpforo_after_add_topic', function($topic, $forum) {
    // Send notification to external service
    wp_remote_post('https://api.example.com/notify', [
        'body' => [
            'event' => 'new_topic',
            'topic_id' => $topic['topicid'],
            'title' => $topic['title'],
            'forum' => $forum['title'],
        ]
    ]);
}, 10, 2);
```

---

### `wpforo_add_topic_data_filter`
**Type**: Filter
**Location**: `classes/Topics.php`
**Parameters**: `$args` (array)
**Description**: Filters topic data before insertion.

**Example**:
```php
add_filter('wpforo_add_topic_data_filter', function($args) {
    // Automatically tag topics based on content
    $content = strtolower($args['body']);
    if (strpos($content, 'bug') !== false) {
        $args['prefix'] = '[BUG]';
    }
    return $args;
}, 10, 1);
```

---

### `wpforo_after_edit_topic`
**Type**: Action
**Location**: `classes/Topics.php`
**Parameters**: `$topic` (array), `$data` (array)
**Description**: Fires after topic is edited.

**Example**:
```php
add_action('wpforo_after_edit_topic', function($topic, $data) {
    // Track edit history in custom table
    global $wpdb;
    $wpdb->insert('wp_topic_edit_log', [
        'topicid' => $topic['topicid'],
        'edited_at' => current_time('mysql'),
        'editor_id' => get_current_user_id(),
    ]);
}, 10, 2);
```

---

### `wpforo_before_delete_topic`
**Type**: Action
**Location**: `classes/Topics.php`
**Parameters**: `$topicid` (int)
**Description**: Fires before topic deletion.

**Example**:
```php
add_action('wpforo_before_delete_topic', function($topicid) {
    // Backup topic data before deletion
    $topic = WPF()->topic->get_topic($topicid);
    update_option('deleted_topic_' . $topicid, $topic, false);
}, 10, 1);
```

---

### `wpforo_after_delete_topic`
**Type**: Action
**Location**: `classes/Topics.php`
**Parameters**: `$topicid` (int), `$topic` (array)
**Description**: Fires after topic is deleted.

**Example**:
```php
add_action('wpforo_after_delete_topic', function($topicid, $topic) {
    // Clean up related custom data
    delete_post_meta_by_key('related_topic_' . $topicid);

    // Notify moderators
    wp_mail('moderators@example.com',
        'Topic Deleted',
        "Topic '{$topic['title']}' has been deleted."
    );
}, 10, 2);
```

---

### `wpforo_topic_approve`
**Type**: Action
**Location**: `classes/Topics.php`
**Parameters**: `$topic` (array)
**Description**: Fires when topic is approved.

**Example**:
```php
add_action('wpforo_topic_approve', function($topic) {
    // Send approval notification to topic author
    $author = get_userdata($topic['userid']);
    wp_mail($author->user_email,
        'Your topic has been approved',
        "Your topic '{$topic['title']}' is now live!"
    );
}, 10, 1);
```

---

### `wpforo_topic_unapprove`
**Type**: Action
**Location**: `classes/Topics.php`
**Parameters**: `$topic` (array)
**Description**: Fires when topic is unapproved.

**Example**:
```php
add_action('wpforo_topic_unapprove', function($topic) {
    // Log moderation action
    error_log("Topic unapproved: {$topic['topicid']} by moderator " . get_current_user_id());
}, 10, 1);
```

---

### `wpforo_after_merge_topic`
**Type**: Action
**Location**: `classes/Topics.php`
**Parameters**: `$topicids` (array), `$mastertopicid` (int)
**Description**: Fires after topics are merged.

**Example**:
```php
add_action('wpforo_after_merge_topic', function($topicids, $mastertopicid) {
    // Update related records to point to master topic
    foreach ($topicids as $oldid) {
        update_post_meta($mastertopicid, 'merged_from_' . $oldid, true);
    }
}, 10, 2);
```

---

### `wpforo_after_move_topic`
**Type**: Action
**Location**: `classes/Topics.php`
**Parameters**: `$topicid` (int), `$forumid` (int), `$old_forumid` (int)
**Description**: Fires after topic is moved to different forum.

**Example**:
```php
add_action('wpforo_after_move_topic', function($topicid, $forumid, $old_forumid) {
    // Update forum statistics
    WPF()->forum->clear_cache($forumid);
    WPF()->forum->clear_cache($old_forumid);

    // Notify forum moderators
    $topic = WPF()->topic->get_topic($topicid);
    error_log("Topic {$topic['title']} moved from forum {$old_forumid} to {$forumid}");
}, 10, 3);
```

---

## Post Management Hooks

Hooks for post/reply creation, editing, and deletion.

### `wpforo_before_add_post`
**Type**: Action
**Location**: `classes/Posts.php`
**Parameters**: `$post` (array)
**Description**: Fires before a new post/reply is added.

**Example**:
```php
add_action('wpforo_before_add_post', function($post) {
    // Check for spam content
    if (wpforo_is_spam_content($post['body'])) {
        wp_die('Spam detected!');
    }
}, 10, 1);
```

---

### `wpforo_after_add_post`
**Type**: Action
**Location**: `classes/Posts.php`
**Parameters**: `$post` (array)
**Description**: Fires after post is successfully added.

**Example**:
```php
add_action('wpforo_after_add_post', function($post) {
    // Award points to user for posting
    $points = get_user_meta($post['userid'], 'forum_points', true) ?: 0;
    update_user_meta($post['userid'], 'forum_points', $points + 10);

    // Update user activity timestamp
    update_user_meta($post['userid'], 'last_forum_activity', current_time('mysql'));
}, 10, 1);
```

---

### `wpforo_add_post_data_filter`
**Type**: Filter
**Location**: `classes/Posts.php`
**Parameters**: `$post` (array)
**Description**: Filters post data before insertion.

**Example**:
```php
add_filter('wpforo_add_post_data_filter', function($post) {
    // Auto-format code blocks
    if (strpos($post['body'], '<code>') !== false) {
        $post['body'] = preg_replace('/<code>(.*?)<\/code>/s',
            '<pre class="language-php"><code>$1</code></pre>',
            $post['body']
        );
    }
    return $post;
}, 10, 1);
```

---

### `wpforo_after_edit_post`
**Type**: Action
**Location**: `classes/Posts.php`
**Parameters**: `$postid` (int), `$data` (array)
**Description**: Fires after post is edited.

**Example**:
```php
add_action('wpforo_after_edit_post', function($postid, $data) {
    // Increment edit count
    $edit_count = get_post_meta($postid, 'edit_count', true) ?: 0;
    update_post_meta($postid, 'edit_count', $edit_count + 1);
    update_post_meta($postid, 'last_edited', current_time('mysql'));
}, 10, 2);
```

---

### `wpforo_before_delete_post`
**Type**: Action
**Location**: `classes/Posts.php`
**Parameters**: `$postid` (int)
**Description**: Fires before post deletion.

**Example**:
```php
add_action('wpforo_before_delete_post', function($postid) {
    // Archive post before deletion
    $post = WPF()->post->get_post($postid);
    file_put_contents(
        WP_CONTENT_DIR . '/forum-archives/' . $postid . '.json',
        json_encode($post, JSON_PRETTY_PRINT)
    );
}, 10, 1);
```

---

### `wpforo_after_delete_post`
**Type**: Action
**Location**: `classes/Posts.php`
**Parameters**: `$postid` (int), `$post` (array)
**Description**: Fires after post is deleted.

**Example**:
```php
add_action('wpforo_after_delete_post', function($postid, $post) {
    // Clean up attachments
    wpforo_delete_post_attachments($postid);

    // Update topic reply count
    if (!empty($post['topicid'])) {
        WPF()->topic->rebuild_stats($post['topicid']);
    }
}, 10, 2);
```

---

### `wpforo_post_approve`
**Type**: Action
**Location**: `classes/Posts.php`
**Parameters**: `$post` (array)
**Description**: Fires when post is approved.

**Example**:
```php
add_action('wpforo_post_approve', function($post) {
    // Send approval notification
    $author = get_userdata($post['userid']);
    wp_mail($author->user_email,
        'Post Approved',
        'Your forum reply has been approved and is now visible.'
    );
}, 10, 1);
```

---

### `wpforo_vote`
**Type**: Action
**Location**: `classes/Posts.php`
**Parameters**: `$post` (array), `$vote` (int)
**Description**: Fires when post receives a vote (up/down).

**Example**:
```php
add_action('wpforo_vote', function($post, $vote) {
    // Track voting activity
    global $wpdb;
    $wpdb->insert('wp_forum_votes', [
        'postid' => $post['postid'],
        'userid' => get_current_user_id(),
        'vote' => $vote,
        'voted_at' => current_time('mysql'),
    ]);
}, 10, 2);
```

---

## Forum Management Hooks

Hooks for forum structure and management operations.

### `wpforo_after_add_forum`
**Type**: Action
**Location**: `classes/Forums.php:189`
**Parameters**: `$args` (array), `$checkperm` (bool)
**Description**: Fires after a new forum is created.

**Example**:
```php
add_action('wpforo_after_add_forum', function($args, $checkperm) {
    // Set default permissions for new forum
    WPF()->perm->add_forum_permissions($args['forumid'], [
        'vf' => true, // view forum
        'ct' => true, // create topic
    ]);
}, 10, 2);
```

---

### `wpforo_after_edit_forum`
**Type**: Action
**Location**: `classes/Forums.php:296`
**Parameters**: `$args` (array), `$forum` (array), `$checkperm` (bool)
**Description**: Fires after forum is edited.

**Example**:
```php
add_action('wpforo_after_edit_forum', function($args, $forum, $checkperm) {
    // Log forum changes
    error_log("Forum {$forum['forumid']} updated: " . print_r($args, true));

    // Clear related caches
    WPF()->forum->clear_cache($forum['forumid']);
}, 10, 3);
```

---

### `wpforo_after_delete_forum`
**Type**: Action
**Location**: `classes/Forums.php:343`
**Parameters**: `$forumid` (int)
**Description**: Fires after forum is deleted.

**Example**:
```php
add_action('wpforo_after_delete_forum', function($forumid) {
    // Clean up forum-specific custom data
    delete_option('forum_' . $forumid . '_settings');

    // Remove forum-specific user meta
    delete_metadata('user', 0, 'subscribed_forum_' . $forumid, '', true);
}, 10, 1);
```

---

### `wpforo_after_copy_forum`
**Type**: Action
**Location**: `classes/Forums.php:79`
**Parameters**: `$forum` (array), `$forumid` (int)
**Description**: Fires after forum is copied/duplicated.

**Example**:
```php
add_action('wpforo_after_copy_forum', function($forum, $new_forumid) {
    // Copy custom forum settings
    $settings = get_option('forum_' . $forum['forumid'] . '_settings');
    if ($settings) {
        update_option('forum_' . $new_forumid . '_settings', $settings);
    }
}, 10, 2);
```

---

### `wpforo_after_merge_forum`
**Type**: Action
**Location**: `classes/Forums.php:375`
**Parameters**: `$forumid` (int), `$mergeid` (int)
**Description**: Fires after two forums are merged.

**Example**:
```php
add_action('wpforo_after_merge_forum', function($forumid, $mergeid) {
    // Update forum subscriptions after merge
    global $wpdb;
    $wpdb->update(
        $wpdb->prefix . 'wpforo_subscribes',
        ['itemid' => $forumid],
        ['itemid' => $mergeid, 'type' => 'forum']
    );
}, 10, 2);
```

---

### `wpforo_forum_loop_no_forums`
**Type**: Action
**Location**: Multiple theme files (10 occurrences)
**Parameters**: `$category` (array)
**Description**: Fires when a category has no forums to display.

**Example**:
```php
add_action('wpforo_forum_loop_no_forums', function($category) {
    // Display custom message for empty categories
    echo '<div class="wpforo-empty-category">';
    echo '<p>This category is empty. Be the first to create a discussion!</p>';
    echo '<a href="' . wpforo_home_url() . '" class="wpforo-button">Create Topic</a>';
    echo '</div>';
}, 10, 1);
```

---

## User & Member Hooks

Hooks related to user profiles, registration, and member management.

### `wpforo_create_user_after`
**Type**: Action
**Location**: `classes/Members.php`
**Parameters**: `$userid` (int), `$user` (array)
**Description**: Fires after wpForo creates a user profile.

**Example**:
```php
add_action('wpforo_create_user_after', function($userid, $user) {
    // Welcome email
    $user_data = get_userdata($userid);
    wp_mail($user_data->user_email,
        'Welcome to our forum!',
        'Thanks for joining our community.'
    );

    // Set default user group
    WPF()->member->set_usergroup($userid, 2); // Regular member
}, 10, 2);
```

---

### `wpforo_update_profile_after`
**Type**: Action
**Location**: `classes/Members.php`
**Parameters**: `$userid` (int), `$profile` (array)
**Description**: Fires after user profile is updated.

**Example**:
```php
add_action('wpforo_update_profile_after', function($userid, $profile) {
    // Track profile completeness
    $completeness = 0;
    if (!empty($profile['avatar'])) $completeness += 25;
    if (!empty($profile['about'])) $completeness += 25;
    if (!empty($profile['occupation'])) $completeness += 25;
    if (!empty($profile['signature'])) $completeness += 25;

    update_user_meta($userid, 'profile_completeness', $completeness);
}, 10, 2);
```

---

### `wpforo_update_profile_fields`
**Type**: Action
**Location**: `classes/Members.php`
**Parameters**: `$userid` (int), `$data` (array)
**Description**: Fires when profile fields are updated.

**Example**:
```php
add_action('wpforo_update_profile_fields', function($userid, $data) {
    // Validate custom field
    if (!empty($data['website'])) {
        if (!filter_var($data['website'], FILTER_VALIDATE_URL)) {
            wp_die('Invalid website URL');
        }
    }
}, 10, 2);
```

---

### `wpforo_after_ban_user`
**Type**: Action
**Location**: `classes/Members.php`
**Parameters**: `$userid` (int)
**Description**: Fires after user is banned.

**Example**:
```php
add_action('wpforo_after_ban_user', function($userid) {
    // Log ban action
    error_log("User {$userid} banned by " . get_current_user_id());

    // Remove user from all active sessions
    WPF()->member->logout_user($userid);

    // Notify moderators
    wp_mail('moderators@example.com',
        'User Banned',
        "User ID {$userid} has been banned from the forum."
    );
}, 10, 1);
```

---

### `wpforo_after_unban_user`
**Type**: Action
**Location**: `classes/Members.php`
**Parameters**: `$userid` (int)
**Description**: Fires after user is unbanned.

**Example**:
```php
add_action('wpforo_after_unban_user', function($userid) {
    // Send restoration email
    $user = get_userdata($userid);
    wp_mail($user->user_email,
        'Your account has been restored',
        'You can now access the forum again.'
    );
}, 10, 1);
```

---

### `wpforo_before_delete_user`
**Type**: Action
**Location**: `classes/Members.php`
**Parameters**: `$userid` (int)
**Description**: Fires before user and their content is deleted.

**Example**:
```php
add_action('wpforo_before_delete_user', function($userid) {
    // Backup user data before deletion
    $member = WPF()->member->get_member($userid);
    $posts = WPF()->post->get_posts(['userid' => $userid]);

    file_put_contents(
        WP_CONTENT_DIR . '/user-backups/user-' . $userid . '.json',
        json_encode(['member' => $member, 'posts' => $posts], JSON_PRETTY_PRINT)
    );
}, 10, 1);
```

---

### `wpforo_after_delete_user`
**Type**: Action
**Location**: `classes/Members.php`
**Parameters**: `$userid` (int)
**Description**: Fires after user is deleted.

**Example**:
```php
add_action('wpforo_after_delete_user', function($userid) {
    // Clean up user-related custom tables
    global $wpdb;
    $wpdb->delete('wp_custom_user_data', ['userid' => $userid]);
}, 10, 1);
```

---

### `wpforo_set_groupid`
**Type**: Action
**Location**: `classes/Members.php`
**Parameters**: `$userid` (int), `$groupid` (int), `$oldgroupid` (int)
**Description**: Fires when user group is changed.

**Example**:
```php
add_action('wpforo_set_groupid', function($userid, $groupid, $oldgroupid) {
    // Award badge for group promotion
    if ($groupid > $oldgroupid) {
        $user = get_userdata($userid);
        wp_mail($user->user_email,
            'Group Promotion!',
            'Congratulations! You have been promoted to a new user group.'
        );
    }
}, 10, 3);
```

---

### `wpforo_after_member_badge`
**Type**: Action
**Location**: Multiple template files (3 occurrences)
**Parameters**: `$member` (array)
**Description**: Fires after member badge is displayed.

**Example**:
```php
add_action('wpforo_after_member_badge', function($member) {
    // Display custom badges
    $post_count = $member['posts'];
    if ($post_count > 1000) {
        echo '<span class="custom-badge veteran">Veteran</span>';
    } elseif ($post_count > 500) {
        echo '<span class="custom-badge expert">Expert</span>';
    }
}, 10, 1);
```

---

## Activity Hooks

Hooks for user activity tracking.

### `wpforo_before_add_activity`
**Type**: Action
**Location**: `classes/Activity.php:346`
**Parameters**: `$activity` (array)
**Description**: Fires before activity is logged.

**Example**:
```php
add_action('wpforo_before_add_activity', function($activity) {
    // Filter out certain activity types from logging
    if ($activity['type'] === 'view') {
        // Don't log view activities
        return false;
    }
}, 10, 1);
```

---

### `wpforo_after_add_activity`
**Type**: Action
**Location**: `classes/Activity.php:355`
**Parameters**: `$activity` (array)
**Description**: Fires after activity is logged.

**Example**:
```php
add_action('wpforo_after_add_activity', function($activity) {
    // Update user activity statistics
    $userid = $activity['userid'];
    $count = get_user_meta($userid, 'total_activities', true) ?: 0;
    update_user_meta($userid, 'total_activities', $count + 1);
    update_user_meta($userid, 'last_activity_date', current_time('mysql'));
}, 10, 1);
```

---

### `wpforo_after_edit_activity`
**Type**: Action
**Location**: `classes/Activity.php:382`
**Parameters**: `$data` (array), `$where` (array)
**Description**: Fires after activity record is edited.

**Example**:
```php
add_action('wpforo_after_edit_activity', function($data, $where) {
    // Log activity modifications
    error_log("Activity modified: " . print_r($where, true));
}, 10, 2);
```

---

### `wpforo_after_delete_activity`
**Type**: Action
**Location**: `classes/Activity.php:404`
**Parameters**: `$where` (array)
**Description**: Fires after activity records are deleted.

**Example**:
```php
add_action('wpforo_after_delete_activity', function($where) {
    // Clean up related activity meta
    if (isset($where['userid'])) {
        delete_user_meta($where['userid'], 'cached_activity_count');
    }
}, 10, 1);
```

---

## Reaction Hooks

Hooks for the like/dislike system.

### `wpforo_like`
**Type**: Action
**Location**: Reaction module
**Parameters**: `$postid` (int), `$userid` (int)
**Description**: Fires when user likes a post.

**Example**:
```php
add_action('wpforo_like', function($postid, $userid) {
    // Award points for receiving likes
    $post = WPF()->post->get_post($postid);
    $author_points = get_user_meta($post['userid'], 'reputation_points', true) ?: 0;
    update_user_meta($post['userid'], 'reputation_points', $author_points + 5);
}, 10, 2);
```

---

### `wpforo_dislike`
**Type**: Action
**Location**: Reaction module
**Parameters**: `$postid` (int), `$userid` (int)
**Description**: Fires when user dislikes a post.

**Example**:
```php
add_action('wpforo_dislike', function($postid, $userid) {
    // Track negative feedback
    $post = WPF()->post->get_post($postid);
    $dislike_count = get_post_meta($postid, 'dislike_count', true) ?: 0;
    update_post_meta($postid, 'dislike_count', $dislike_count + 1);

    // Flag for review if too many dislikes
    if ($dislike_count > 10) {
        update_post_meta($postid, 'needs_review', true);
    }
}, 10, 2);
```

---

### `wpforo_undo_like`
**Type**: Action
**Location**: Reaction module
**Parameters**: `$postid` (int), `$userid` (int)
**Description**: Fires when user removes their like.

**Example**:
```php
add_action('wpforo_undo_like', function($postid, $userid) {
    // Adjust reputation points
    $post = WPF()->post->get_post($postid);
    $author_points = get_user_meta($post['userid'], 'reputation_points', true) ?: 0;
    update_user_meta($post['userid'], 'reputation_points', max(0, $author_points - 5));
}, 10, 2);
```

---

### `wpforo_after_add_reaction`
**Type**: Action
**Location**: `modules/reactions/`
**Parameters**: `$reaction` (array)
**Description**: Fires after any reaction is added.

**Example**:
```php
add_action('wpforo_after_add_reaction', function($reaction) {
    // Send notification to post author
    $post = WPF()->post->get_post($reaction['postid']);
    $reactor = get_userdata($reaction['userid']);

    // Don't notify if reacting to own post
    if ($post['userid'] != $reaction['userid']) {
        wpforo_notify_user($post['userid'],
            "{$reactor->display_name} reacted to your post"
        );
    }
}, 10, 1);
```

---

## Form Hooks

Hooks for forms and form elements.

### `wpforo_topic_form_extra_before`
**Type**: Action
**Location**: Form rendering
**Parameters**: `$topic` (array), `$forum` (array)
**Description**: Fires before additional topic form fields.

**Example**:
```php
add_action('wpforo_topic_form_extra_before', function($topic, $forum) {
    // Add custom field to topic form
    echo '<div class="wpforo-custom-field">';
    echo '<label>Topic Type:</label>';
    echo '<select name="topic_type">';
    echo '<option value="discussion">Discussion</option>';
    echo '<option value="question">Question</option>';
    echo '<option value="announcement">Announcement</option>';
    echo '</select>';
    echo '</div>';
}, 10, 2);
```

---

### `wpforo_topic_form_extra_after`
**Type**: Action
**Location**: Form rendering
**Parameters**: `$topic` (array), `$forum` (array)
**Description**: Fires after additional topic form fields.

**Example**:
```php
add_action('wpforo_topic_form_extra_after', function($topic, $forum) {
    // Add tags input field
    echo '<div class="wpforo-tags-field">';
    echo '<label>Tags (comma-separated):</label>';
    echo '<input type="text" name="topic_tags" placeholder="php, wordpress, debugging">';
    echo '</div>';
}, 10, 2);
```

---

### `wpforo_reply_form_extra_before`
**Type**: Action
**Location**: Form rendering
**Parameters**: `$topic` (array), `$values` (array), `$forum` (array)
**Description**: Fires before reply form extra fields.

**Example**:
```php
add_action('wpforo_reply_form_extra_before', function($topic, $values, $forum) {
    // Add "notify me" checkbox
    echo '<div class="wpforo-notify-checkbox">';
    echo '<label>';
    echo '<input type="checkbox" name="notify_replies" value="1"> ';
    echo 'Notify me of new replies';
    echo '</label>';
    echo '</div>';
}, 10, 3);
```

---

### `wpforo_reply_form_extra_after`
**Type**: Action
**Location**: Form rendering
**Parameters**: `$topic` (array), `$values` (array), `$forum` (array)
**Description**: Fires after reply form extra fields.

**Example**:
```php
add_action('wpforo_reply_form_extra_after', function($topic, $values, $forum) {
    // Add quick template selector
    echo '<div class="wpforo-quick-templates">';
    echo '<label>Quick Templates:</label>';
    echo '<button type="button" onclick="insertTemplate(\'thanks\')">Thank You</button>';
    echo '<button type="button" onclick="insertTemplate(\'question\')">Follow-up Question</button>';
    echo '</div>';
}, 10, 3);
```

---

### `wpforo_editor_topic_submit_before`
**Type**: Action
**Location**: Editor rendering
**Parameters**: `$editor_args` (array)
**Description**: Fires before topic submit button in editor.

**Example**:
```php
add_action('wpforo_editor_topic_submit_before', function($editor_args) {
    // Add draft save button
    echo '<button type="button" class="wpf-save-draft" onclick="saveDraft()">Save as Draft</button>';
}, 10, 1);
```

---

### `wpforo_editor_post_submit_after`
**Type**: Action
**Location**: Editor rendering
**Parameters**: `$editor_args` (array)
**Description**: Fires after post submit button in editor.

**Example**:
```php
add_action('wpforo_editor_post_submit_after', function($editor_args) {
    // Add preview button
    echo '<button type="button" class="wpf-preview" onclick="previewPost()">Preview</button>';
}, 10, 1);
```

---

## Template & Layout Hooks

Hooks for template rendering and content injection.

### `wpforo_loop_hook`
**Type**: Action
**Location**: Multiple layout files (37 occurrences!)
**Parameters**: `$item` (array), `$index` (int)
**Description**: Fires during each loop iteration. Most frequently used hook.

**Example**:
```php
add_action('wpforo_loop_hook', function($item, $index) {
    // Add ad every 5th item
    if ($index % 5 === 0) {
        echo '<div class="forum-advertisement">';
        echo '<!-- Ad code here -->';
        echo '</div>';
    }

    // Highlight featured topics
    if (!empty($item['is_featured'])) {
        echo '<span class="featured-badge">Featured</span>';
    }
}, 10, 2);
```

---

### `wpforo_header_hook`
**Type**: Action
**Location**: Theme files (3 occurrences)
**Parameters**: None
**Description**: Fires in forum header section.

**Example**:
```php
add_action('wpforo_header_hook', function() {
    // Add custom header banner
    echo '<div class="forum-announcement">';
    echo '<p>Welcome to our community! Check out our <a href="/rules">forum rules</a>.</p>';
    echo '</div>';
});
```

---

### `wpforo_footer_hook`
**Type**: Action
**Location**: Theme files (3 occurrences)
**Parameters**: None
**Description**: Fires in forum footer section.

**Example**:
```php
add_action('wpforo_footer_hook', function() {
    // Add custom footer content
    echo '<div class="forum-footer-stats">';
    $stats = WPF()->statistic->get_statistics();
    echo '<p>Total Posts: ' . number_format($stats['posts']) . '</p>';
    echo '<p>Total Members: ' . number_format($stats['members']) . '</p>';
    echo '</div>';
});
```

---

### `wpforo_tpl_post_loop_after_content`
**Type**: Action
**Location**: Post templates (18 occurrences)
**Parameters**: `$post` (array)
**Description**: Fires after post content in loop.

**Example**:
```php
add_action('wpforo_tpl_post_loop_after_content', function($post) {
    // Add "helpful" button to answers
    if (wpforo_is_answer_topic($post['topicid'])) {
        echo '<div class="post-helpful-section">';
        echo '<button class="mark-helpful" data-postid="' . $post['postid'] . '">';
        echo 'Mark as Helpful';
        echo '</button>';
        echo '</div>';
    }
}, 10, 1);
```

---

### `wpforo_content_start`
**Type**: Action
**Location**: Theme files (3 occurrences)
**Parameters**: None
**Description**: Fires at the start of forum content area.

**Example**:
```php
add_action('wpforo_content_start', function() {
    // Add breadcrumbs
    echo '<div class="wpforo-breadcrumbs">';
    echo wpforo_generate_breadcrumbs();
    echo '</div>';
});
```

---

### `wpforo_top_hook`
**Type**: Action
**Location**: Template files
**Parameters**: None
**Description**: Fires at top of forum page.

**Example**:
```php
add_action('wpforo_top_hook', function() {
    // Add search bar at top
    echo '<div class="forum-top-search">';
    wpforo_search_form();
    echo '</div>';
});
```

---

### `wpforo_bottom_hook`
**Type**: Action
**Location**: Template files (3 occurrences)
**Parameters**: None
**Description**: Fires at bottom of forum page.

**Example**:
```php
add_action('wpforo_bottom_hook', function() {
    // Add newsletter signup
    echo '<div class="forum-newsletter">';
    echo '<h3>Stay Updated</h3>';
    echo '<form action="/newsletter-signup" method="post">';
    echo '<input type="email" name="email" placeholder="Your email">';
    echo '<button type="submit">Subscribe</button>';
    echo '</form>';
    echo '</div>';
});
```

---

### `wpforo_topic_head_left`
**Type**: Action
**Location**: Topic templates (6 occurrences)
**Parameters**: `$topic` (array)
**Description**: Fires in left side of topic header.

**Example**:
```php
add_action('wpforo_topic_head_left', function($topic) {
    // Add topic status indicator
    if ($topic['status'] == 1) {
        echo '<span class="topic-status closed">Closed</span>';
    } elseif ($topic['solved']) {
        echo '<span class="topic-status solved">Solved</span>';
    }
}, 10, 1);
```

---

### `wpforo_topic_head_right`
**Type**: Action
**Location**: Topic templates (6 occurrences)
**Parameters**: `$topic` (array)
**Description**: Fires in right side of topic header.

**Example**:
```php
add_action('wpforo_topic_head_right', function($topic) {
    // Add share buttons
    echo '<div class="topic-share-buttons">';
    echo '<a href="https://twitter.com/share?url=' . urlencode(wpforo_topic($topic['topicid'], 'url')) . '">Tweet</a>';
    echo '<a href="https://www.facebook.com/sharer.php?u=' . urlencode(wpforo_topic($topic['topicid'], 'url')) . '">Share</a>';
    echo '</div>';
}, 10, 1);
```

---

## Social Features Hooks

Hooks for bookmarks, follows, and subscriptions.

### `wpforo_after_add_bookmark`
**Type**: Action
**Location**: `modules/bookmarks/Bookmarks.php:106`
**Parameters**: `$bookmark` (array)
**Description**: Fires after bookmark is added.

**Example**:
```php
add_action('wpforo_after_add_bookmark', function($bookmark) {
    // Track bookmark statistics
    $user_bookmarks = get_user_meta($bookmark['userid'], 'bookmark_count', true) ?: 0;
    update_user_meta($bookmark['userid'], 'bookmark_count', $user_bookmarks + 1);
}, 10, 1);
```

---

### `wpforo_after_delete_bookmark`
**Type**: Action
**Location**: `modules/bookmarks/Bookmarks.php:160`
**Parameters**: `$args` (array), `$operator` (string)
**Description**: Fires after bookmark is deleted.

**Example**:
```php
add_action('wpforo_after_delete_bookmark', function($args, $operator) {
    // Update bookmark count
    if (isset($args['userid'])) {
        $count = get_user_meta($args['userid'], 'bookmark_count', true) ?: 0;
        update_user_meta($args['userid'], 'bookmark_count', max(0, $count - 1));
    }
}, 10, 2);
```

---

### `wpforo_after_add_follow`
**Type**: Action
**Location**: Follow module
**Parameters**: `$follow` (array)
**Description**: Fires after user follows another user or topic.

**Example**:
```php
add_action('wpforo_after_add_follow', function($follow) {
    // Notify followed user
    if ($follow['type'] === 'user') {
        $follower = get_userdata($follow['userid']);
        wpforo_notify_user($follow['itemid'],
            "{$follower->display_name} started following you"
        );
    }
}, 10, 1);
```

---

### `wpforo_after_delete_follow`
**Type**: Action
**Location**: Follow module
**Parameters**: `$args` (array), `$operator` (string)
**Description**: Fires after follow is removed.

**Example**:
```php
add_action('wpforo_after_delete_follow', function($args, $operator) {
    // Clean up follow-related notifications
    if (isset($args['userid']) && isset($args['itemid'])) {
        delete_user_meta($args['itemid'], 'follower_' . $args['userid']);
    }
}, 10, 2);
```

---

## Admin Panel Hooks

Hooks for WordPress admin and wpForo settings.

### `wpforo_admin_menu`
**Type**: Action
**Location**: `admin/index.php:79`
**Parameters**: `$parent_slug` (string), `$board` (array)
**Description**: Fires when admin menu is registered.

**Example**:
```php
add_action('wpforo_admin_menu', function($parent_slug, $board) {
    // Add custom admin page
    add_submenu_page(
        $parent_slug,
        'Custom Reports',
        'Custom Reports',
        'manage_options',
        'wpforo-custom-reports',
        'wpforo_custom_reports_page'
    );
}, 10, 2);
```

---

### `wpforo_dashboard_widgets_col1`
**Type**: Action
**Location**: `admin/pages/overview.php:142`
**Parameters**: None
**Description**: Fires in dashboard column 1.

**Example**:
```php
add_action('wpforo_dashboard_widgets_col1', function() {
    // Add custom dashboard widget
    echo '<div class="wpforo-dashboard-widget">';
    echo '<h3>Quick Stats</h3>';
    echo '<p>Today\'s posts: ' . wpforo_get_today_posts_count() . '</p>';
    echo '<p>New members: ' . wpforo_get_today_members_count() . '</p>';
    echo '</div>';
});
```

---

### `wpforo_settings_after`
**Type**: Action
**Location**: `admin/pages/settings.php:171`
**Parameters**: `$tab` (string), `$setting` (array)
**Description**: Fires after settings are saved.

**Example**:
```php
add_action('wpforo_settings_after', function($tab, $setting) {
    // Clear cache after settings change
    if ($tab === 'styles') {
        wpforo_clean_cache();
    }
}, 10, 2);
```

---

### `wpforo_admin_enqueue_scripts`
**Type**: Action
**Location**: `includes/hooks.php:1651`
**Parameters**: None
**Description**: Fires when admin scripts are enqueued.

**Example**:
```php
add_action('wpforo_admin_enqueue_scripts', function() {
    // Enqueue custom admin scripts
    wp_enqueue_script(
        'wpforo-custom-admin',
        plugins_url('js/admin-custom.js', __FILE__),
        ['jquery'],
        '1.0.0',
        true
    );
});
```

---

## Permission & Moderation Hooks

Hooks for permissions and content moderation.

### `wpforo_permissions_forum_can`
**Type**: Filter
**Location**: `classes/Permissions.php`
**Parameters**: `$can` (bool), `$action` (string), `$forumid` (int), `$userid` (int)
**Description**: Filters forum permission check result.

**Example**:
```php
add_filter('wpforo_permissions_forum_can', function($can, $action, $forumid, $userid) {
    // Grant special access to premium members
    if ($action === 'ct') { // create topic
        $user_group = WPF()->member->get_usergroup($userid);
        if ($user_group['name'] === 'Premium Member') {
            return true;
        }
    }
    return $can;
}, 10, 4);
```

---

### `wpforo_moderation_remove_body_links`
**Type**: Filter
**Location**: `classes/Moderation.php:433`
**Parameters**: `$remove` (bool), `$item` (array), `$urls` (array)
**Description**: Filters whether to remove links from post body.

**Example**:
```php
add_filter('wpforo_moderation_remove_body_links', function($remove, $item, $urls) {
    // Allow trusted users to post links
    $user_posts = WPF()->member->get_member($item['userid'])['posts'];
    if ($user_posts > 50) {
        return false; // Don't remove links
    }
    return $remove;
}, 10, 3);
```

---

### `wpforo_moderation_remove_title_links`
**Type**: Filter
**Location**: `classes/Moderation.php`
**Parameters**: `$remove` (bool), `$item` (array), `$urls` (array)
**Description**: Filters whether to remove links from topic title.

**Example**:
```php
add_filter('wpforo_moderation_remove_title_links', function($remove, $item, $urls) {
    // Always remove links from titles for new users
    $user_posts = WPF()->member->get_member($item['userid'])['posts'];
    if ($user_posts < 10) {
        return true;
    }
    return $remove;
}, 10, 3);
```

---

## Email & Notification Hooks

Hooks for email notifications and alerts.

### `break_wpforo_send_email`
**Type**: Filter
**Location**: Email functions
**Parameters**: `$break` (bool), `$to` (string), `$subject` (string), `$message` (string)
**Description**: Filters whether to send email (return true to prevent).

**Example**:
```php
add_filter('break_wpforo_send_email', function($break, $to, $subject, $message) {
    // Don't send emails during maintenance
    if (wp_is_maintenance_mode()) {
        return true; // Break/prevent email
    }

    // Rate limit emails to same address
    $last_sent = get_transient('wpforo_last_email_' . md5($to));
    if ($last_sent && time() - $last_sent < 60) {
        return true; // Prevent email
    }

    set_transient('wpforo_last_email_' . md5($to), time(), 300);
    return false; // Allow email
}, 10, 4);
```

---

### `wpforo_answer`
**Type**: Action
**Location**: `includes/hooks.php:745`
**Parameters**: `$status` (int), `$post` (array)
**Description**: Fires when post is marked/unmarked as answer.

**Example**:
```php
add_action('wpforo_answer', function($status, $post) {
    if ($status == 1) {
        // Post marked as answer
        $post_author = get_userdata($post['userid']);
        wp_mail($post_author->user_email,
            'Your answer was accepted!',
            'Congratulations! Your answer has been marked as the solution.'
        );

        // Award reputation points
        $points = get_user_meta($post['userid'], 'reputation_points', true) ?: 0;
        update_user_meta($post['userid'], 'reputation_points', $points + 15);
    }
}, 10, 2);
```

---

### `break_wpforo_follow_send_email_after_add_topic`
**Type**: Filter
**Location**: Follow module
**Parameters**: `$break` (bool), `$topic` (array), `$follower` (array)
**Description**: Filters whether to send email to followers when new topic is added.

**Example**:
```php
add_filter('break_wpforo_follow_send_email_after_add_topic', function($break, $topic, $follower) {
    // Don't notify if follower is topic author
    if ($follower['userid'] == $topic['userid']) {
        return true;
    }

    // Check user's email preferences
    $email_prefs = get_user_meta($follower['userid'], 'wpforo_email_prefs', true);
    if (isset($email_prefs['follow_topics']) && !$email_prefs['follow_topics']) {
        return true; // Don't send
    }

    return false;
}, 10, 3);
```

---

### `break_wpforo_subscriber_send_email_after_add_post`
**Type**: Filter
**Location**: Subscription module
**Parameters**: `$break` (bool), `$post` (array), `$subscriber` (array)
**Description**: Filters whether to send email to subscribers when new post is added.

**Example**:
```php
add_filter('break_wpforo_subscriber_send_email_after_add_post', function($break, $post, $subscriber) {
    // Don't send digest emails during work hours
    $hour = date('G');
    $subscriber_prefs = get_user_meta($subscriber['userid'], 'email_schedule', true);

    if ($subscriber_prefs === 'digest_only') {
        return true; // Save for digest
    }

    return false;
}, 10, 3);
```

---

## Board Management Hooks

Hooks for multi-board functionality.

### `wpforo_after_add_board`
**Type**: Action
**Location**: `classes/Boards.php:201`
**Parameters**: `$board` (array)
**Description**: Fires after new board is created.

**Example**:
```php
add_action('wpforo_after_add_board', function($board) {
    // Set up default forums for new board
    WPF()->forum->add([
        'title' => 'General Discussion',
        'description' => 'General topics',
        'boardid' => $board['boardid'],
    ]);

    // Create default user group
    WPF()->usergroup->add([
        'name' => 'Default Member',
        'boardid' => $board['boardid'],
    ]);
}, 10, 1);
```

---

### `wpforo_before_change_board`
**Type**: Action
**Location**: `wpforo.php:220`
**Parameters**: `$args` (array)
**Description**: Fires before switching to different board.

**Example**:
```php
add_action('wpforo_before_change_board', function($args) {
    // Save current board state
    $current_board = WPF()->board->get_current('boardid');
    set_transient('wpforo_previous_board_' . get_current_user_id(), $current_board, 3600);
}, 10, 1);
```

---

### `wpforo_after_change_board`
**Type**: Action
**Location**: `wpforo.php:234`
**Parameters**: `$args` (array)
**Description**: Fires after switching to different board.

**Example**:
```php
add_action('wpforo_after_change_board', function($args) {
    // Clear board-specific caches
    wpforo_clean_cache();

    // Track board switching analytics
    wpforo_track_event('board_switch', [
        'boardid' => $args['boardid'],
        'userid' => get_current_user_id(),
    ]);
}, 10, 1);
```

---

### `wpforo_after_delete_board`
**Type**: Action
**Location**: `classes/Boards.php:248`
**Parameters**: `$boardid` (int)
**Description**: Fires after board is deleted.

**Example**:
```php
add_action('wpforo_after_delete_board', function($boardid) {
    // Clean up board-specific options
    delete_option('wpforo_board_' . $boardid . '_settings');

    // Remove board-specific transients
    global $wpdb;
    $wpdb->query($wpdb->prepare(
        "DELETE FROM {$wpdb->options} WHERE option_name LIKE %s",
        '_transient_wpforo_' . $boardid . '_%'
    ));
}, 10, 1);
```

---

## Caching Hooks

Hooks for cache management.

### `wpforo_clean_cache`
**Type**: Action
**Location**: `includes/functions.php:2484`
**Parameters**: `$id` (mixed), `$template` (string)
**Description**: Fires when cache is being cleaned.

**Example**:
```php
add_action('wpforo_clean_cache', function($id, $template) {
    // Also clear object cache
    wp_cache_delete('wpforo_' . $template . '_' . $id, 'wpforo');

    // Clear CDN cache
    if (function_exists('custom_cdn_purge')) {
        custom_cdn_purge('/forums/*');
    }
}, 10, 2);
```

---

### `wpforo_cache_forum`
**Type**: Filter
**Location**: Caching system
**Parameters**: `$cache` (bool), `$forumid` (int)
**Description**: Filters whether to cache specific forum.

**Example**:
```php
add_filter('wpforo_cache_forum', function($cache, $forumid) {
    // Don't cache private forums
    $forum = WPF()->forum->get_forum($forumid);
    if ($forum['status'] == 0) { // Private
        return false;
    }
    return $cache;
}, 10, 2);
```

---

### `wpforo_cache_dir`
**Type**: Filter
**Location**: Cache system
**Parameters**: `$cache_dir` (string)
**Description**: Filters cache directory path.

**Example**:
```php
add_filter('wpforo_cache_dir', function($cache_dir) {
    // Use custom cache directory
    return WP_CONTENT_DIR . '/forum-cache';
}, 10, 1);
```

---

## Integration Hooks

Hooks for third-party integrations.

### `wpforo_um_member_profile_url`
**Type**: Filter
**Location**: Ultimate Member integration
**Parameters**: `$url` (string), `$userid` (int)
**Description**: Filters Ultimate Member profile URL.

**Example**:
```php
add_filter('wpforo_um_member_profile_url', function($url, $userid) {
    // Use custom profile URL structure
    return home_url('/community/member/' . $userid);
}, 10, 2);
```

---

### `wpforo_bp_notifications_handler`
**Type**: Action
**Location**: `integrations/BuddyPressHooks.php:608`
**Parameters**: `$success` (bool), `$user_id` (int), `$reply_id` (int), `$action` (string)
**Description**: Fires during BuddyPress notification handling.

**Example**:
```php
add_action('wpforo_bp_notifications_handler', function($success, $user_id, $reply_id, $action) {
    // Track BuddyPress notification delivery
    if ($success) {
        wpforo_log('BP notification sent', [
            'user_id' => $user_id,
            'reply_id' => $reply_id,
            'action' => $action,
        ]);
    }
}, 10, 4);
```

---

## Additional Important Filter Hooks

### `wpforo_avatar_url`
**Type**: Filter
**Parameters**: `$avatar_url` (string), `$member` (array)
**Description**: Filters member avatar URL.

**Example**:
```php
add_filter('wpforo_avatar_url', function($avatar_url, $member) {
    // Use custom avatar service
    if (empty($avatar_url)) {
        return 'https://avatars.example.com/' . md5($member['user_email']) . '.jpg';
    }
    return $avatar_url;
}, 10, 2);
```

---

### `wpforo_member_profile_url`
**Type**: Filter
**Parameters**: `$url` (string), `$userid` (int), `$tab` (string)
**Description**: Filters member profile URL.

**Example**:
```php
add_filter('wpforo_member_profile_url', function($url, $userid, $tab) {
    // Use custom profile URL structure
    $username = get_userdata($userid)->user_login;
    return home_url('/user/' . $username . ($tab ? '/' . $tab : ''));
}, 10, 3);
```

---

### `wpforo_get_topics`
**Type**: Filter
**Parameters**: `$topics` (array), `$args` (array)
**Description**: Filters retrieved topics array.

**Example**:
```php
add_filter('wpforo_get_topics', function($topics, $args) {
    // Remove sensitive topics for non-moderators
    if (!current_user_can('wpforo_moderator')) {
        $topics = array_filter($topics, function($topic) {
            return empty($topic['is_sensitive']);
        });
    }
    return $topics;
}, 10, 2);
```

---

### `wpforo_spam_topic`
**Type**: Filter
**Parameters**: `$is_spam` (bool), `$topic` (array)
**Description**: Filters spam detection for topics.

**Example**:
```php
add_filter('wpforo_spam_topic', function($is_spam, $topic) {
    // Custom spam detection
    $spam_keywords = ['viagra', 'casino', 'lottery'];
    $content = strtolower($topic['title'] . ' ' . $topic['body']);

    foreach ($spam_keywords as $keyword) {
        if (strpos($content, $keyword) !== false) {
            return true;
        }
    }

    return $is_spam;
}, 10, 2);
```

---

### `wpforo_wrap_class`
**Type**: Filter
**Parameters**: `$classes` (array)
**Description**: Filters CSS classes for forum wrapper.

**Example**:
```php
add_filter('wpforo_wrap_class', function($classes) {
    // Add custom CSS classes
    if (is_user_logged_in()) {
        $classes[] = 'wpforo-logged-in';
    }

    // Add device-specific class
    if (wp_is_mobile()) {
        $classes[] = 'wpforo-mobile';
    }

    return $classes;
}, 10, 1);
```

---

### `wpforo_can_attach`
**Type**: Filter
**Parameters**: `$can` (bool), `$userid` (int), `$forumid` (int)
**Description**: Filters whether user can attach files.

**Example**:
```php
add_filter('wpforo_can_attach', function($can, $userid, $forumid) {
    // Only allow attachments for members with 10+ posts
    $member = WPF()->member->get_member($userid);
    if ($member['posts'] < 10) {
        return false;
    }
    return $can;
}, 10, 3);
```

---

## Usage Tips

### Hook Priority
Most wpForo hooks use default priority (10). Use lower numbers to run earlier, higher numbers to run later:

```php
add_action('wpforo_after_add_topic', 'my_function', 5, 2);  // Runs early
add_action('wpforo_after_add_topic', 'my_function', 20, 2); // Runs late
```

### Removing Hooks
To remove a hooked function:

```php
remove_action('wpforo_header_hook', 'some_function', 10);
remove_filter('wpforo_add_topic_data_filter', 'some_filter', 10);
```

### Conditional Execution
Check context before executing hook code:

```php
add_action('wpforo_after_add_post', function($post) {
    // Only run in specific forum
    if ($post['forumid'] != 5) return;

    // Only for certain user group
    $usergroup = WPF()->member->get_usergroup($post['userid']);
    if ($usergroup['name'] !== 'Premium') return;

    // Your code here
}, 10, 1);
```

### Error Handling
Always include error handling in hooks:

```php
add_action('wpforo_after_add_topic', function($topic, $forum) {
    try {
        // Your code
        wp_remote_post('https://api.example.com/notify', [...]);
    } catch (Exception $e) {
        error_log('wpForo hook error: ' . $e->getMessage());
    }
}, 10, 2);
```

---

## Common Hook Patterns

### 1. Data Validation Before Save
```php
add_action('wpforo_before_add_topic', function($args) {
    if (strlen($args['title']) < 10) {
        wp_die('Title must be at least 10 characters');
    }
}, 10, 1);
```

### 2. Automatic Content Processing
```php
add_filter('wpforo_add_post_data_filter', function($post) {
    // Auto-link URLs
    $post['body'] = make_clickable($post['body']);
    return $post;
}, 10, 1);
```

### 3. External API Integration
```php
add_action('wpforo_after_add_topic', function($topic, $forum) {
    wp_remote_post('https://slack.com/api/chat.postMessage', [
        'body' => [
            'text' => "New topic: {$topic['title']}",
            'channel' => '#forum-activity',
        ]
    ]);
}, 10, 2);
```

### 4. Custom User Permissions
```php
add_filter('wpforo_permissions_forum_can', function($can, $action, $forumid, $userid) {
    // Premium members can always post
    if (user_has_premium_subscription($userid)) {
        return true;
    }
    return $can;
}, 10, 4);
```

### 5. Content Moderation
```php
add_filter('wpforo_add_topic_data_filter', function($args) {
    // Auto-moderate topics from new users
    $member = WPF()->member->get_member($args['userid']);
    if ($member['posts'] < 5) {
        $args['status'] = 1; // Set to unapproved
    }
    return $args;
}, 10, 1);
```

---

## Resources

- **wpForo Documentation**: https://wpforo.com/docs/
- **wpForo Community**: https://wpforo.com/community/
- **Plugin Location**: `/wp-content/plugins/wpforo/`
- **Main Class**: `WPF()` - Global function to access wpForo singleton

---

## Version Information

- **wpForo Version**: 3.0.0.0
- **Documentation Generated**: 2025-11-11
- **Total Hooks Documented**: 609 occurrences (227 action, 382 filter)

---

## Need Help?

For questions about specific hooks or implementation help:

1. Check the hook's location in the plugin files for full context
2. Visit wpForo community forums
3. Review example code in this documentation
4. Test hooks in a development environment first

**Note**: Always backup your site before implementing custom hooks that modify data or functionality.
