ElasticSearch 索引 read-only 状态恢复

最近在做 ES 压测，当磁盘快满时，所有索引会进入 read-only 状态。数据无法继续写入，索引无法关闭，只能删除索引。ES [官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/disk-allocator.html)中对此种情况进行了说明.

如果使用 ES 的默认设置，ES 为了保持节点可用，设置了几个存储的安全值。分别为：

```yaml
cluster.routing.allocation.disk.watermark.low: 默认 85% 当达到时，replica 不再写入 cluster.routing.allocation.disk.watermark.high: 默认 90% 当达到时，shards 会尝试写入其他节点
cluster.routing.allocation.disk.watermark.flood_stage: 默认 95% 当达到时，所有索引变为 readonly状态
```

所以遇到 read-only 状态应该先检查磁盘是否快满了，此处可用通过 ES 日志查看。在释放空间后，需要手动将每个索引的状态修改过来，可以通过

```shell
curl -XPUT 'localhost:9200/twitter/_settings?pretty' -H 'Content-Type: application/json' -d'
{
  "index.blocks.read_only_allow_delete": null
}
'

```

修改状态

不过我还是建议使用 postman

如果你的索引较多，这将是一个痛苦的过程，可以通过编写脚本将此过程自动化

如果在释放空间前恢复索引状态，在一段时间后 index 会再次变为 read-only ，因为 ES 每隔一定时间会再次检查磁盘情况，在 6.x 版本中是30s。