# Bai2

This library implements the [Bai2 standard][bai2], as per its official
specification.

[bai2]: http://www.bai.org/Libraries/Site-General-Downloads/Cash_Management_2005.sflb.ashx

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'bai2'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install bai2

## Usage

`BaiFile` is the main class in gem.

```ruby
# Parse a file:
file = Bai2::BaiFile.parse('file.bai2')
# Parse data:
file = Bai2::BaiFile.new(string_data)

puts file.sender, file.receiver

# e.g. filter for groups relevant to your organization, iterate:
file.groups.filter {|g| g.destination == YourOrgId }.each do |group|

  # groups have accounts
  group.accounts.each do |account|

    puts account.customer, account.currency_code

    # summaries are arrays of hashes
    puts account.summaries.inspect

    # accounts have transactions

    # e.g. print all debits
    account.transactions.filter(&:debit?).each do |debit|

      # transactions have string amounts, too
      puts debit.amount

      # transaction types are represented by an informative hash:
      puts debit.type
      # => {
      #  code:        451,
      #  transaction: :debit,
      #  scope:       :detail,
      #  description: "ACH Debit Received",
      # }

      puts debit.text
    end

    # e.g. print sum of all credits
    sum = account.transactions \
      .filter(&:credit?) \
      .map(&:amount) \
      .map {|a| BigDecimal(a) } \
      .reduce(&:+)
    puts sum.inspect

  end
end
```

## Caveats

In `lib/bai2/integrity.rb`, we perform integrity checks mandated by the Bai2
standard. In our experience, the spec and bank’s real implementations differ on
how sums are calculated. It’s hard to tell if this an industry-wide trend, or
just an SVB quirk. I would love to hear how other banks do this. GitHub Issues
with more information on this would be greatly appreciated.

```ruby
# Check sum vs. summary + transaction sums
actual_sum = self.transactions.map(&:amount).reduce(0, &:+) \
  #+ self.summaries.map {|s| s[:amount] }.reduce(0, &:+)
  # TODO: ^ there seems to be a disconnect between what the spec defines
  # as the formula for the checksum and what SVB implements...
```


## Contributing

1. Fork it ( https://github.com/venturehacks/bai2/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
