# coding: utf-8
require 'qiita'
require 'formatador'
require 'logger'

BEST_BEFORE="<!-- too_old -->\n> **この記事は最終更新から1年以上経過しています。** 気をつけてね。\n"
PERMANENT="<!-- permanent -->\n"
@client = Qiita::Client.new(access_token: ENV['QIITA_TOKEN'])
@logger = Logger.new($stdout)

desc "show all items"
task :show do
  all_items = return_all_items
  all_items.sort! { |a, b| a['since_last_update'] <=> b['since_last_update'] }
  # ["rendered_body", "body", "coediting", "created_at", "id", "private", "tags", "title", "updated_at", "url", "user"]
  # Formatador.display_table(all_items, ['created_at', 'updated_at', 'since_last_update', 'tagged', 'title', 'url'])
  Formatador.display_compact_table(all_items, ['created_at', 'since_last_update', 'tagged', 'permanent', 'id', 'title', 'url'])
end

## PATCH
# {
#   "body": "# Example",
#   "coediting": false,
#   "private": false,
#   "tags": [
#     {
#       "name": "example",
#       "versions": [
#         "0.0.1"
#       ]
#     }
#   ],
#   "title": "hello world"
# }

task :preview do
  all_items = return_all_items
  tester = all_items[-4]
  item = set_header_to_item(tester)
  patch_header_to_item(item)
end

desc "put warning header to all items"
task :all do
  all_items = return_all_items
  all_items.each do |item|
    updater = set_header_to_item(item)
    @logger.info updater["id"] + " will be update." if updater
    patch_header_to_item(updater)
  end
end

def return_all_items
  all_items = []
  page = 1

  items = @client.list_user_items(ENV['QIITA_USER'], page: page)

  while items.next_page_url do
    all_items.concat(items.body)
    page += 1
    items = @client.list_user_items(ENV['QIITA_USER'], page: page)
  end

  all_items.concat(items.body)

  # Time.strptime('2015-04-03T15:38:50+09:00', '%Y-%m-%dT%H:%M:%S')
  all_items.map do |i|
    i['since_last_update']  = ( (Time.now - Time.strptime(i['updated_at'], '%Y-%m-%dT%H:%M:%S')) / 60 / 60 / 24 ).to_i
    i['tagged'] = i['body'].start_with?(BEST_BEFORE)
    i['permanent'] = i['body'].start_with?(PERMANENT)
    i
  end
end

def set_header_to_item(item)
  @logger.info item["id"] + ": " + item["since_last_update"].to_s + " days."
  return false if item["tagged"]
  return false if item["permanent"]
  return false if item["since_last_update"] < 365
  item["body"] = BEST_BEFORE +  item["body"]
  item
end

def patch_header_to_item(item)
  if item
    res = @client.update_item(item["id"], {
      body:       item["body"],
      coediting:  item["coediting"],
      private:    item["private"],
      tags:       item["tags"],
      title:      item["title"]
    })
    @logger.info res
  end
end

