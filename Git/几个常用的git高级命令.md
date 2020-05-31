Git blame

Git blame 可以查询每一行代码的 commit ID、提交者和提交日期。

$ git blame nova/api/openstack/compute/servers.py
 
5b866f3a nova/api/openstack/v2/servers.py        (Kevin L. Mitchell    2012-01-09 13:13:08 -0600 776)   @wsgi.response(202)
5b866f3a nova/api/openstack/v2/servers.py        (Kevin L. Mitchell    2012-01-09 13:13:08 -0600 777)   @wsgi.serializers(xml=FullServerTemplate)
5b866f3a nova/api/openstack/v2/servers.py        (Kevin L. Mitchell    2012-01-09 13:13:08 -0600 778)   @wsgi.deserializers(xml=CreateDeserializer)
5b866f3a nova/api/openstack/v2/servers.py        (Kevin L. Mitchell    2012-01-09 13:13:08 -0600 779)   def create(self, req, body):
cc642ff1 nova/api/openstack/compute/servers.py     (Alex Meade        2012-04-15 21:44:15 -0400 780)     """Creates a new server for a given user."""
d1ad73ee nova/api/openstack/compute/servers.py     (Mark McLoughlin     2012-09-12 12:50:53 +0100 781)     if not self.is_valid_body(body, 'server'):
5b866f3a nova/api/openstack/v2/servers.py        (Kevin L. Mitchell

Git show

确切的说，Git show 并非是一个高级命令，只是它经常配合 git blame，从而查询整个 patch。

$ git show 5b866f3a
 
...
 
diff --git a/nova/api/openstack/common.py b/nova/api/openstack/common.py
index 5def03f..e96c42a 100644
--- a/nova/api/openstack/common.py
+++ b/nova/api/openstack/common.py
@@ -334,6 +334,21 @@ def get_networks_for_instance(context, instance):
   return networks
 
 
+class MetadataDeserializer(wsgi.MetadataXMLDeserializer):
\+  def deserialize(self, text):
\+    dom = minidom.parseString(text)
\+    metadata_node = self.find_first_child_named(dom, "metadata")
\+    metadata = self.extract_metadata(metadata_node)
 
...

Git reflog

Git reflog 记录了 git 某个分支的每次操作，通常用来恢复误操作影响的数据。

$ git reflog
 
1dcfb0f HEAD@{0}: reset: moving to 1dcfb0f
d5640a9 HEAD@{1}: checkout: moving from test to master
1dcfb0f HEAD@{2}: checkout: moving from icehouse to test
 
$ git reset d5640a9

Git cherry-pick

Git cherry-pick 可从其它分支抓取 commit 合入当前分支中，常用于从 upstream 合入 patch。

$ git cherry-pick -x commit_id

Git rebase

Git rebase 无疑是最为复杂、最难以理解，当然也是非常强大的命令，常用于合并分支，可以简单的理解为一系列的 git cherry-pick，它具有以下功能。

• 合并分支

• 重新修改 Commit，如合并 commit，调整 commit 的顺序。

Git rebase 用法请见 Git Book，但是使用时，请注意以下准则：

The golden rule of git rebase is to never use it on public branches.